## Overview (API)

Powerpage open a window with MS WebBrowser Control. When HTML page is loaded, Powerpage will import ``powerpage.js`` 
to initialize ``pb`` javascript object to provide Powerpage interface.

HTML page may via the following channel to talk to main program

1. recommented call by by javascript: ``pb.apiFunction()``, e.g. pb.run('notepad.exe')
2. by url: ``&lt;a href="pb://command/parameters">Text&lt;/a>`` or ``window.location = "pb://command/parameters"``
3. by change title: ``document.title = "pb://command/parameters"``

Powerpage will interpret and execute the command, and pass the result to HTML page by calling js function ``pb.router( result, type, cmd)``

for example:  
  
ps: ``If call within Powerpage, and the following links turn ^^red^^, then it is clickable ^^Powerpage API call^^.``

* Run notepad.exe to edit powerpage.ini -> ``javascript:pb.run('notepad.exe powerpage.ini')`` or ``pb://run/notepad.exe powerpage.ini``
* Run SQL1 and callback showData() -> ``javascript:pb.callback('showData').db.query(sql1)`` or ``pb://callback/showData/db/query/@sql1``  
* Run update SQL2 -> ``javascript:pb.db.execute(sql3)`` or ``pb://db/execute/@sql3``
* Call About window -> ``javascript:pb.window('w_about')`` or ``pb://window/w_about``
 
   
## Global Features  ##
  
``Callback, Prompt/Confirm, @JsVar and Secure Protocol`` are supported in all commands. 
 
   
-----------------------------------------------------------------------------------
### Callback ###

Every command may specify callback function. If not specify, program will call onCallback() as default.

#### Syntax

* url-protocol: **pb://callback**/{callback-function-name}/command 
* javascript : **javascript:pb.callback({callback-function-name})**.apiFunction()

#### Samples

* `pb://callback/mycallback/run/mstsc.exe` 
* `javascript:pb.callback('mycallback').run('mstsc.exe')` 
* `javascript:pb.callback('mycallback').run('cmd=notepad.exe,style=wait')` 

#### Source Code (powerpage.js)

Powerpage will call `pb.rounter()` in powerpage.js to route to callback-function() by ``window[name]( result, type, url )``,

* callback-function should be a window-level function which is accessible by `window[{callback-function-name}]` in javascript.
* if callback-function is specified but not found, program will show alert message. 
* if callback not specify, pb.router() will call onCallback() as default.

~~~ powerpage.js
//=== router function. call from Powerbuilder, divert to callback function
pb.router = function ( name, result, type, url ) {
  try {
    if (typeof window[name] === "function") {
        window[name]( result, type, url );
    } else if (name) {
        alert( 'callback function ' + name + '() not found!\n\n type:' + type + '\n cmd: ' + url 
               + '\n function: '+name + '\n result: \n\n' + result )
    } else if (typeof onCallback === "function") {
        onCallback( result, type, url );
    }
  } catch (e) {
    alert( 'Error in callback! \n\n Name: ' + name + '\nMessage:' + e.message )
  }  
}
~~~
  
 
-----------------------------------------------------------------------------------
### Prompt/Confirm

Prompt for confirmation, then run command
 
#### Syntax

* url-protocol: **pb://?{prompt-message}?/**command 
* javascript : **javascript:pb.confirm({prompt-message})**.apiFunction()
* javascript : **javascript:pb.prompt({prompt-message})**.apiFunction()

#### Samples

* `javascript:pb.confirm('Open notepad?').run('notepad.exe')`
* `pb://?Open notepad?/run/notepad.exe` 

#### Source Code (powerbuilder)

~~~ powerbuilder
//=== powerbuilder.event titlechange()
// handle prompt message
if mid(text,4,4) = '///?' and pos(text,'?/',7) > 0 then	
	if messagebox( 'Confirmation', mid( text, 8, pos(text,'?/',7) - 7 ), Question!, YesNo! ) <> 1 then 
		return
	else
		text = left(text,5) + mid( text, pos(text,'?/',7) + 2 )
	end if
end if     
~~~
 
  
-----------------------------------------------------------------------------------
### @js Variable Replacement

Use javascript variable to store long string, and pass to Powerpage interface. 
For string parameter start with "@' (i.e. `@{var}`) will be replace with the value of variable.

#### Samples

* javascript: pb.db.query(`sql1`) 
* javascript: pb.db.query(`'@sql1'`)
* url-protocol: pb://sql/query/`@sql1``

#### Source Code (powerbuilder + powerpage.js)

For string paramter start with '@', Powerbuilder call javascript function pb('@var') to retrieve its value. 

~~~ powerbuilder
string function of_get_string( string as_text ) 
  // load string from js variable
  if left(as_text,1)='@' then 
  	as_text = this.object.document.script.pb( mid(as_text,2) )	
  end if
  
  // handle some special characters
  as_text = of_replaceall( as_text, '&gt;', '>' )
  as_text = of_replaceall( as_text, '&lt;', '<' )
	
return as_text
~~~

in powerpage.js, declare function pb('@var') to return js variable value.

~~~ powerpage.js
// pb main function, pb('varname') = js.varname, pb('#div') = getElementById
var pb = function (n) { 
  return n[0]=='#'? document.getElementById(n.substr(1)) : window[n]; 
}
~~~
 
    
-----------------------------------------------------------------------------------
### Secured protocol

Once use ``Secured`` protocol, Powerpage will prompt user login by windows account.

#### Syntax

* url-protocol: **ps://**command 
* javascript : **javascript:pb.secure()**.apiFunction()

#### Samples
 
*  `javascript:pb.secure().run('resmon.exe')`
* ps-protocol: `ps://run/resmon.exe` 
 
#### Source Code (powerbuilder)

~~~ powerbuilder
//=== powerbuilder.event titlechange() 
// handle ps:// security mode
if lower(left(text,5)) = 'ps://' and trim(gnv_app.is_login_user) = '' then
  open(w_login)
  if trim(gnv_app.is_login_user) = '' then 
    return
  end if
  gnv_app.of_microhelp( 'window user login. id=' + gnv_app.is_login_user )
end if  
~~~  
 
    
           
## Run / Shell 
 
Run will call Wscript.run() to run DOS command,
Shell command will be diverted to `shell32.dll ->ShellExecuteW()`
   
   
-----------------------------------------------------------------------------------
### pb.run( command )
### pb.run( cmd, path, style, callback ) 

Run a program, e.g. resmon.exe

#### Syntax

* url-protocol: **pb://run/**{command} 
* javascript : **javascript:pb.run**({command})
* javascript : **javascript:pb.run**( {cmd}, {path}, {style}, callback )
* {command} := cmd={cmd},path={path},style={style}
* {cmd}  := dos command 
* {path} := path to run the command
* {style} := [ normal | min | max | hide | wait ] 

#### Samples

* ``javascript:pb.run('resmon.exe')`` run resmon.exe
* ``javascript:pb.run('notepad.exe powerpage.html')`` run notepad to edit powerpage.html
* pb-protocol: ``pb://run/notepad.exe powerpage.html`` run notepad to edit powerpage.html
* ``javascript:pb.run('powerpage.exe', 'c:\app')`` run powerpage at c:\app
* ``javascript:pb.run('notepad.exe powerpage.html','.','max+wait','alert')`` edit edit powerpage.html and show status 
 
#### Source Code (powerbuilder+powerpage.js)

~~~ powerbuilder
//## [20210611] pb://run/folder=?,cmd=?,style=[min|max|normal|hide]+wait
if is_type = 'run' then
	
	ls_opts = mid(is_command,5) 
	ls_path = of_get_keyword( ls_opts, 'path', ',', of_get_keyword( ls_opts, 'folder', ',', ls_null ) )
	ls_run = of_get_keyword( ls_opts, 'cmd', ',', ls_null )
	ls_style = of_get_keyword( ls_opts, 'style', ',', 'normal' ) 

	if pos(lower(ls_style),'normal')>0 then
		ll_show = 1
	elseif pos(lower(ls_style),'min')>0 then
		ll_show = 2
	elseif pos(lower(ls_style),'max')>0 then
		ll_show = 3
	elseif pos(lower(ls_style),'hide')>0 then
		ll_show = 0
	else
		ll_show = 1
	end if

	if ls_run > ' ' then
		changedirectory( ls_path )
		ll_rtn = of_wsh_run( ls_run, ll_show, pos(ls_style,'wait')>0 )
		changedirectory( gnv_app.is_currentPath )
	else
		ll_rtn = run( mid( is_command, 5 ) )
	end if
	
	return event ue_callback( as_callback, '{ "status":'+string(ll_rtn)+'}' )	
	
end if 
~~~


~~~ powerpage.js
pb.run = function ( cmd, path, style, callback ) { 
  if (arguments.length==1) {
    pb.submit( 'run', cmd ) 
  } else {
    var ls_opt = 'cmd=' + cmd + (path? ',path='+path : '' ) + (style? ',style='+style : '' )
    pb.submit( 'run', ls_opt, callback )
  } 
}
~~~


-----------------------------------------------------------------------------------
### pb.shell(command)
### pb.shell(action, file, parm, path, show, callback)

* Description: calling window.shell object to "Open/Run/Print" an file. 
* pb-protocol: `` pb://shell/{command}`` or `` pb://shell/file={file},path={path},show={show},action={action}``

```
pb.shell = function ( action, file, parm, path, show, callback ) { 
  if (arguments.length==1) {
    pb.submit( 'shell', action )
  } else {
    var ls_opt = 'file=' + file + (action? ',action='+action : '' ) + (parm? ',parm='+parm : '' )
    pb.submit( 'shell', ls_opt + (path? ',path='+path : '' ) + (show? ',show='+show : '' ), callback )
  } 
}
```

##### Arguments

* {action}  // aciton := [ open | print | runas ]
* {file}  // file name  
* {path}  // path of the file
* {parm}  // parameters passed to the file
* {show}  // show (or style) := [ normal | min | max | hide | wait ] 


##### Samples 

* ``pb.shell('calc.exe')`` // run calc.exe
* ``pb.shell('open','calc.exe')``  // call calc.exe using shell-open (same as start.exe [file])
* ``pb.shell('run', 'c:\\app\\powerpage.exe')`` // shell-run program (similar to runat())
* ``pb://shell/run/c:\powerpage\powerpage.exe`` // shell-run program (similar to runat())    

-----------------------------------------------------------------------------------
### pb.sendkeys(keys)

call sendkeys() function of Wscript Shell to send keystrokes.
 
keys := /js={js-function}/run={cmd}/title={goto Title}/s={delay}/ms={delay ms}/Keystrokes

##### Samples 

* ``javascript:pb.sendkeys('/run=notepad.exe/title=Untitled - Notepad/s=2/this is a test')``
* ``pb://sendkeys/run=notepad.exe/title=Untitled - Notepad/s=2/this is a test``
* ``javascript:pb.sendkeys(keys)``  
* ``javascript:pb.sendkeys('@keys')`` or ``pb://sendkeys/@keys`` pass keystrokes by variable

#### Source Code (powerbuilder)

~~~ powerbuilder
//# [20210604] ck, call Wscript to process sendkeys and commands
//# keys := run=command/go=title/js=jsFunciton/s=1000/&#47;&sol;
//#=============================================

string ls_cmd
long  ll_ms, ll_cpu

if not iole_wsh.IsAlive() then
	ii_connect = iole_wsh.ConnectToNewObject("WScript.Shell")
	if ii_connect<0 then return ii_connect
end if  

do 
	ls_cmd = trim( of_gettoken( as_keys, '/' ) )
	ls_cmd = of_replaceall( of_replaceall( ls_cmd, '&#47;', '/' ), '&sol;', '/' )
	
	// run shell command
	if left(ls_cmd,4) = 'run=' then 
		iole_wsh.run( mid( ls_cmd, 5 ) )
		
	// goto window by title	
	elseif left(ls_cmd,6) = 'title=' then 
		iole_wsh.AppActivate( mid( ls_cmd, 7 ) )
		
	// call js function	 (call pb.router to activate js command)
	elseif left(ls_cmd,3) = 'js=' then
		this.object.document.script.pb.router( mid( ls_cmd, 4 ), as_keys, 'sendkeys', 'call' )

	// wait for seconds	
	elseif left(ls_cmd,2) = 's=' then 
		sleep( long( mid( ls_cmd, 3 ) ) )
		
	// wait for milliseconds	
	elseif left(ls_cmd,3) = 'ms=' then
		ll_cpu = cpu()
		ll_ms = long( mid( ls_cmd, 4 ) )
		do
			yield()
		loop while (cpu() - ll_cpu) < ll_ms
		
	// send keystrokes.	
	else
		iole_wsh.SendKeys(ls_cmd)
		
	end if

loop while trim(as_keys)>''

return 1
~~~


   
## Database Accessibility
  
Query from database, execute sql or stored procedure.

  
-----------------------------------------------------------------------------------
### pb.db.query( sql, callback )

Run SQL select statement, and return result-set in json string format.

``{ colCount:?, rowCount:?, column: [ col1, col2, ...], data:[ [row1,...], [row2,...], ...] }``

* may use alias ``pb.db.select( sql, callback )`` for same feature.
* callback function may use ``rs = JSON.parse(result)`` to get result-set for process.

#### Samples

* ``javascript:pb.db.query('select CategoryID, CategoryName from Categories')`` run select SQL
* ``javascript:pb.db.select('select date(), now()')`` select current date/time
* ``javascript:pb.db.query(sql1)``  run SQL query from js variable sql1  
* ``javascript:pb.db.query('@sql1')``  run SQL query using js variable replacement  
* ``pb://db/query/@sql1``  run SQL query using js variable sql1   


#### Source (powerbuilder)

~~~ powerbuilder
PUBLIC FUNCTION string of_sql_to_json (string as_sql)

datastore lds
long i, j, ll_colCount, ll_rowCount
string ls_syntax, ls_error, ls_json, ls_colName, ls_colType[], ls_row

// conenct db
if not of_connectdb(3) then return '{ "error": "database not connected" }'

// dw syntax
sqlca.autocommit=TRUE
ls_syntax = sqlca.syntaxfromsql( as_sql, 'Style(Type=grid)', ls_error )
sqlca.autocommit=FALSE
if ls_error > '' then
	messagebox( 'Invlaid SQL Syntax', ls_error )
	return '{ "error": "' + ls_error + '" }'
end if

lds = create datastore
lds.create(ls_syntax)

lds.settransobject(sqlca)
lds.retrieve()

ll_colCount = integer( lds.Object.DataWindow.Column.Count )
ll_rowCount = lds.rowcount()
ls_json   = '{  "colCount":' + string(ll_colCount) + ' ~r~n '
ls_json += ' , "rowCount":' + string(ll_rowCount) + ' ~r~n '
ls_json += ' , "columns": [ ~r~n ' 

for i = 1 to ll_colCount
	ls_colName = lds.Describe( '#'+String(i)+'.name')
	ls_colName = lds.Describe( ls_colname+'_t.text' )
	ls_colType[i] = lds.Describe( '#'+String(i)+'.coltype')
	if i>1 then ls_json += ', '
	ls_json += '"' + of_replaceall( of_replaceall(ls_colName,'"','' ), '~r~n', ' ' ) + '"' 
next

ls_json += '] ~r~n, "data": [ ~r~n ' 

for i = 1 to ll_rowcount
	ls_row = ' [ '
	for j=1 to ll_colCount
		if j>1 then ls_row += ' , '
		if left( ls_colType[j], 5 ) = 'char('  then
			ls_row += of_default( '"' +  of_js_string( lds.getitemstring( i, j ) ) + '"', 'null' )
		elseif  ls_colType[j] = 'date' then
			ls_row += of_default( '"' + string( lds.getitemdate( i, j ), 'yyyy/mm/dd' ) + '"', 'null' )
		elseif  ls_colType[j] = 'datetime' or  ls_colType[j] = 'timestamp' then
			ls_row += of_default( '"' + string( lds.getitemdatetime( i, j ), 'yyyy/mm/dd hh:mm:ss' ) + '"', 'null' )
		elseif ls_colType[j] = 'time' then
			ls_row += of_default( '"' + string( lds.getitemtime( i, j ), 'hh:mm:ss' ) + '"', 'null' )
		else
			ls_row += of_default( string( lds.GetItemDecimal( i, j ) ), 'null' )
		end if
	next
	//ls_row = of_replaceall( ls_row, '\"','&quot;' )
	//ls_row = of_replaceall( ls_row, '~r~n','\n' )
	ls_json += of_replaceall( ls_row, '~r~n','\n' ) + ' ]  ,~r~n'
next

return left( ls_json, len(ls_json) - 4 ) + ' ~r~n ] } '
~~~

   
-----------------------------------------------------------------------------------
### pb.db.html( sql, callback )

run SQL select statement, and return result set in HTML table format.

~~~
<table class="pb-table">
  <tr> <th>ColName1</th> <th>..</th> <th>..</th> </tr>
  <tr> <td>Row1-1<td> <td>Row1-2<td> <td>Row1-3<td> </tr>
  <tr> <td>Row2-1<td> <td>Row2-2<td> <td>Row2-3<td> </tr>
</table>
~~~

#### Samples

* ``javascript:pb.db.html('select CategoryID, CategoryName from Categories')`` run select SQL
* ``javascript:pb.db.html('select date(), now()')`` select current date/time
* ``javascript:pb.db.html(sql1)``  run SQL query from js variable sql1  
* ``javascript:pb.db.html('@sql2')``  run SQL query using js variable replacement  
* ``pb://db/html/@sql2``  run SQL query using js variable sql1  


-----------------------------------------------------------------------------------
### pb.db.execute( sql, callback )

execute SQL statement, and return result in json string format.

* may use alias: ``pb.db.update( sql, callback )`` instead.
* return json string `{status:1, message:"SQL Executed Sucessfully!"}` if done 
* return json string `{status:1, error:?, message:"error message"}` if error 

#### Samples

* ``javascript:pb.db.execute(sql3)`` Execute update statement 
* ``javascript:pb.db.update(sql3)`` Execute update statement 
* ``pb://db/execute/@sql3`` Execute update statement 

#### Source (powerbuilder)

~~~
PUBLIC FUNCTION string of_sql_execute (string as_sql)
string ls_msg

// run sql
of_connectdb(3)
EXECUTE IMMEDIATE :as_sql using sqlca; 

// return
if sqlca.sqlcode<>0 then 
	gnv_app.of_microhelp( '[Error] code:'+string(sqlca.sqlcode)+', message:' + of_default(sqlca.sqlerrtext,'null')  )
	ROLLBACK using sqlca;
	return '{ "status": -1, "error":' + string(sqlca.sqlcode)+ ', "message": "' + of_default(sqlca.sqlerrtext,'null') + '" }' 
end if

COMMIT USING sqlca;

gnv_app.of_microhelp( '>> SQL Executed Sucessfully!' )
RETURN '{ "status": 1, "message": "SQL Executed Sucessfully!" }' 
~~~

-----------------------------------------------------------------------------------
### pb.db.prompt( sql, callback )

prompt SQL statement for confirmation, then execute SQL statement. 

* may use alias: ``pb.db.confirm( sql, callback )`` instead.
* return json string `{status:1, message:"SQL Executed Sucessfully!"}` if done 
* return json string `{status:-1, error:?, message:"error message"}` if error 
* return json string `{status:0, message:"SQL execution cancelled" }` if cancelled 

#### Samples

* ``javascript:pb.db.prompt(sql3)`` confirm then execute update statement 
* ``javascript:pb.db.confirm(sql3)`` confirm then execute update statement 
* ``pb://db/prompt/@sql3`` confirm then execute update statement 


-----------------------------------------------------------------------------------
### to-do: pb.db.queryById( id, args[], callback )  
### to-do: pb.db.executeById( id, args[], callback )  
  

                                                                  
## File Accessibility

Powerpage provide the following function to access file system.   
  
----------------------------------------------------------------------------------
### pb.file.copy(from, to, callback)

copy file {from} to {to}, then activate callback function. 

#### Samples

* ``javascript:pb.file.copy( 'powerpage.ini', 'newfile.ini' )`` Copy powerpage.ini to newfile.ini 
* ``pb://file/copy/powerpage.ini/newfile.ini`` Copy powerpage.ini to newfile.ini 

#### Source (powerbuilder)

~~~ powerbuilder
// pb://file/copy/{SourceFile}/{targetFile}
if as_action='copy' then
	
	ls_text = as_parm
	ls_name = of_gettoken(ls_text,'/')
	ll_rtn = FileCopy ( ls_name, ls_text, true )
	
	if ll_rtn > 0 then
		return gnv_app.of_replaceall( '{ "status": 1, "message":"copy file ' + ls_name + ' to ' + ls_text + '" } ', '\', '\\' )
	else
		return gnv_app.of_replaceall( '{ "status": '+string(ll_rtn)+', "message":"failed to copy file ' + ls_name + '" } ', '\', '\\' )
	end if
	
end if
~~~


----------------------------------------------------------------------------------
### pb.file.move(from, to, callback)

move file {from} to {to}, then activate callback function. 

#### Samples

* ``javascript:pb.file.move( 'newfile.ini', 'another.ini' )`` Move newfile.ini to another.ini 
* ``pb://file/move/newfile.ini/another.ini`` Move newfile.ini to another.ini

#### Source (powerbuilder)

~~~ powerbuilder
// pb://file/move/{SourceFile}/{targetFile}
if as_action='move' then
	
	ls_text = as_parm
	ls_name = of_gettoken(ls_text,'/')
	ll_rtn = FileMove ( ls_name, ls_text )
	
	if ll_rtn > 0 then
		return  gnv_app.of_replaceall( '{ "status": 1, "message":"move file ' + ls_name + ' to ' + ls_text + '" } ', '\', '\\' )
	else
		return gnv_app.of_replaceall( '{ "status": '+string(ll_rtn)+', "message":"failed to move file ' + ls_name + '" } ', '\', '\\' )
	end if
	
end if
~~~


----------------------------------------------------------------------------------
### pb.file.exists(file, callback)

check file existance, return 'true/false' if found/notFound. 

#### Samples

* ``javascript:pb.file.exists('newfile.ini')``  Check file existance of newfile.ini
* ``javascript:pb.file.exists('another.ini','alert')``  Check file existance of another.ini
* ``pb://file/exists/another.ini`` Check file existance of another.ini

#### Source (powerbuilder)

~~~ powerbuilder
// pb://file/exists/{FileName}
if as_action = 'exists' then
	if fileexists( as_parm ) then
		return 'true'
	else
		return 'false'
	end if
end if
~~~


----------------------------------------------------------------------------------
### pb.file.read(file, callback)

read text file and callback with result (in string format). 
return empty string if not found.

#### Samples
 
* ``javascript:pb.file.read('another.ini','alert')`` read file and show by alert()
* ``pb://file/read/another.ini`` read file and show by default callback()

#### Source (powerbuilder)

~~~ powerbuilder
// pb://file/read/{FileName}
if as_action = 'read' then
	
	ll_file = fileopen( as_parm, TextMode!, Read! )
	if ll_file<=0 then return ''

	filereadex( ll_file, ls_result )
	fileclose(ll_file)
	return ls_result
	
end if
~~~


----------------------------------------------------------------------------------
### pb.file.write(file, text, callback)

write text to file, and callback.

* return `{ status:{count}, message:"write to {file}  successfully." }` if done
* return `{ status:{-code}, message:"failed to open file {file}" }` if open error
* return '{ status:-9, message: "failed to write file {file}" } ` if write error

#### Samples
 
* ``javascript:pb.file.write('another.ini',sql1)`` write sql1 to file
* ``javascript:pb.file.write('another.ini','@sql2')`` write sql1 to file (recommended for long string)
* ``pb://file/write/another.ini/@sql1`` write sql1 to file 
* ``pb://callback/alert/file/write/another.ini/@sql2`` write sql1 to file 

#### Source (powerbuilder)

~~~ powerbuilder
// pb://file/append/{FileName}/{Content|@var}
if as_action='append' or as_action='write' then
	
	ls_text = as_parm
	ls_name = of_gettoken( ls_text, '/' )
	ls_text  = of_get_string( ls_text )
	
	if as_action='write' then
		filedelete(ls_name)
		ll_file = fileopen( ls_name, TextMode!, Write!, LockReadWrite!, Replace!, EncodingUTF8! )
	else
		ll_file = fileopen( ls_name, TextMode!, Write!, LockReadWrite!, Append! )
	end if
	
	if ll_file<=0 then 
		event ue_pb_error( '[File Error] canot open file.', as_action+'/' +as_parm )
		return  '{ "status": '+string(ll_file)+' , "message":"failed to open file ' + gnv_app.of_replaceall(ls_name,'\','\\') + '" } '
	end if
	
	if filewriteex( ll_file, ls_text )<0 then
		fileclose(ll_file)
		event ue_pb_error( '[File Error] canot write file.', as_action+'/' +as_parm )
		return '{ "status": -9, "message":"failed to write file ' + gnv_app.of_replaceall(ls_name,'\','\\') + '" } '
	end if
	
	fileclose(ll_file)
	return '{ "status":' + string(len(ls_text)) + ', "message":"write to ' + gnv_app.of_replaceall(ls_name,'\','\\') + ' successfully." } '

end if
~~~


----------------------------------------------------------------------------------
### pb.file.append(file, text, callback)

Append text to file, and callback.

* return `{ status:{count}, message:"write to {file}  successfully." }` if done
* return `{ status:{-code}, message:"failed to open file {file}" }` if open error
* return '{ status:-9, message: "failed to write file {file}" } ` if write error
* source code refer to pb.file.write(file, text, callback)

#### Samples
 
* ``javascript:pb.file.append('another.ini',sql2)`` write sql2 to file
* ``javascript:pb.file.append('another.ini','@sql2')`` write sql2 to file (recommended for long string)
* ``javascript:pb.file.read('another.ini','alert')`` read another.ini, show by alert()
* ``pb://file/append/another.ini/@sql3`` append @sql3 to another.ini 

  
----------------------------------------------------------------------------------
### pb.file.delete(file, callback)

delete file, and callback.

#### Samples
 
* ``javascript:pb.file.delete('another.ini')`` delete file another.ini
* ``javascript:pb.file.delete('newfile.ini')`` delete file another.ini 
* ``pb://file/delete/another.ini`` delete file another.ini  

#### Source (powerbuilder)

~~~ powerbuilder
// pb://file/delete/{FileName}
if as_action='delete' then
	if filedelete( as_parm  ) then
		return  '{ "status": 1, "message":"file ' + gnv_app.of_replaceall(as_parm,'\','\\') + ' deleted" } '
	else
		return '{ "status": -1, "message":"failed to delete file ' + gnv_app.of_replaceall(as_parm,'\','\\') + '!" } '
	end if
end if
~~~
  

----------------------------------------------------------------------------------
### pb.file.opendialog(filter, callback)

open a dialog to select file for **open**. 

**Filter**

A string whose value is a text description of the files to include in the list box 
and the file mask that you want to use to select the displayed files (for example, *.* or *.exe). 
The format for filter is: `description,*.ext

To specify multiple filter patterns for a single display string, use a semicolon to separate the patterns, 
for example: "Graphic Files (*.bmp;*.gif;*.jpg;*.jpeg),*.bmp;*.gif;*.jpg;*.jpeg"

The default is:"All Files (*.*),*.*"

**Return**
* return ``{ status:1, path: {path}, file: {file} }`` if file selected
* return ``{ status:0 }`` if file not selected

**Samples**
 
* ``javascript:pb.file.opendialog('Markdown (*.md),*.md')`` open dialog for markdown file 
* ``javascript:pb.file.opendialog('Javascript (*.js),*.js')`` select js files 
* ``pb://file/opendialog/Ini (*.ini),*.ini`` open dialog for ini   

**Source** (powerbuilder)

~~~ powerbuilder
if as_action='opendialog' then
	ll_rtn = GetFileOpenName ( '', ls_path, ls_file, '', as_parm )
	if ll_rtn >0 then 
		return gnv_app.of_replaceall( '{ "status": 1, "path":"' + ls_path + '", "file":"' + ls_file + '" } ', '\', '\\' )
	else
		return '{ "status": '+string(ll_rtn)+' } '
	end if
end if	
~~~
  

-----------------------------------------------------
### pb.file.savedialog(filter, callback)

Open a dialog to select file for **save**. 

**Filter**

A string whose value is a text description of the files to include in the list box 
and the file mask that you want to use to select the displayed files (for example, *.* or *.exe). 
The format for filter is: `description,*.ext`

To specify multiple filter patterns for a single display string, use a semicolon to separate the patterns, 
for example: `Graphic Files (*.bmp;*.gif;*.jpg;*.jpeg),*.bmp;*.gif;*.jpg;*.jpeg`

The default is: `All Files (*.*),*.*`

**Return**
* return ``{ status:1, path: {path}, file: {file} }`` if file selected
* return ``{ status:0 }`` if file not selected

**Samples**
 
* ``javascript:pb.file.savedialog('Markdown (*.md),*.md')`` open dialog for markdown file 
* ``javascript:pb.file.savedialog('Javascript (*.js),*.js')`` select js files 
* ``pb://file/savedialog/Ini (*.ini),*.ini`` open dialog for ini   

**Source** (powerbuilder)

~~~ powerbuilder
// pb://file/select/extension
if as_action='select' or as_action='savedialog' then
	ll_rtn = GetFileSaveName ( 'Select File', ls_path, ls_file, '', as_parm )
	if ll_rtn >0 then 
		return gnv_app.of_replaceall( '{ "status": 1, "path":"' + ls_path + '", "file":"' + ls_file + '" } ', '\', '\\' )
	else
		return '{ "status": '+string(ll_rtn)+' } '
	end if
end if	
~~~
  
    
   
## Work with Powerbuilder
  
This program is developed using Powerbuilder 10.5. 
It is supported to call powerbuilder customized window/function from html. 

  
-----------------------------------------------------
### pb.window(win, args, callback)

Open Powerbuilder window with parameters.

**Samples**

* `javascript: pb.window('w_about')` open about dialog  
* `pb://window/w_about` open about dialog  
* `javascript: pb.window('w_power_dialog','left=50,height=500,url=https://google.com')` open google in popup dialog   
* `pb://window/w_power_dialog/top=20,width=800,url=https://google.com` open google in popup dialog  

**Source** (powerbuilder)

~~~ powerbuilder
//## pb://window/{windowName}[/{parm}] => Call PB dialog window (with parm)
if is_type='window' then
	setnull(message.stringparm)
	if trim(ls_parm)>'' then
		ll_rtn = openwithparm( lw_win, ls_parm, ls_name )	
	else
		ll_rtn = open( lw_win, ls_name )	
	end if
	if message.stringparm > '' then
		return event ue_callback( as_callback, '{ "status":'+string(ll_rtn)+', "return":"' + message.stringparm + '"}' )
	else
		return event ue_callback( as_callback, '{ "status":'+string(ll_rtn)+'}' )
	end if
end if
~~~


-----------------------------------------------------
### pb.func(name, args, callback)

call Powerbuilder user-defined function. This is advanced feature for extented PB library. 

1. define extented PB library in ini. e.g. `extLibrary=myapp.pbl`
2. code global function `f_functions( name, parm )` 
3. handle ( name, parm ) within the function, e.g. run SQL, or call other funcitons. 

**Source** (powerbuilder)

~~~ powerbuilder     
//## pb://function/{name}/{parameter} => dynamic call f_functions() to handle. it shall be defined in ext pb library
if is_type='func' then
	TRY	
		ls_result = dynamic f_functions(ls_name, ls_parm)
	CATCH (runtimeerror er)
		ls_result = '{ "status":-1, "message":"function not found - ' + ls_name + '()" } ' 
	END TRY	
	return event ue_callback( as_callback, ls_result )
end if
~~~     
     
## Misc Features

-----------------------------------------------------
### Session / Global Variables

``pb.session`` serves as session object sharing information between different pages. initialized by ``powerpage.ini``, e.g.

          [session]
          var1 = name: PowerPage 
          var2 = version: version 0.38, updated on 2021/05/05 
          var3 = remarks: this is a test message
          var4 = user: { "id":"admin", "role":"admin", "group":"IT" } 

to update session variables, may use protocol ``pb://session/remarks/new content`` or by javascript ``pb.session('remarks','new content')``   


-----------------------------------------------------
### pb.popup(url,callback) 

Pop HTML Dialog within pwoerpage

To open url in popup dialog (share session info), use protocol ``pb://popup/height=400,url=dialog.html`` or javascript ``pb.popup('width=800,url=dialog.html')`` 

To popup dialog with callback, by protocol ``pb://callback/mycallback/popup/height=400,url=dialog.html`` or 
by javascript ``pb.popup('width=500,url=dialog.html','mycallback') or pb.callback('mycallback').popup('width=500,url=dialog.html')``

-----------------------------------------------------
### pb.print( opt, callback )

* Print (default, with prompt) => ``pb://print`` or js ``pb.print()``
* Print without prompt => ``pb://print.now`` or js ``pb.print('now')``
* Print Preview => ``pb://print/preview`` or js ``pb.print('preview')`` 
* Print Setup => ``pb://print/setup`` or js ``pb.print('setup')``


-----------------------------------------------------
### pb.pdf( opt, parm, callback )  


-----------------------------------------------------
### pb.pdf( opt, parm, callback )  


-----------------------------------------------------
### pb.spider


-----------------------------------------------------
## Modification History

* 2021/05/11  initial
* 2021/05/31  add print commands
* 2021/09/30  update document using markdown
* 2021/10/05  update document (run/shell)
* 2021/11/03  update document (db/file/pb functions)


