festival-text-to-speech-service
===============================
###Overview
This sample web service calls upon a Festival server to do synthesize text into audio. Communication between the browser, web service, and Festival service is as follows:

1) Browser sends HTTP request to the Java-based web service

2) Web service acts as a client and calls the Festival speech server

3) Festival server returns audio as a byte array

4) Web service handles the byte array audio and returns it to the browser

The Festival TTS server returns all audio as some form of uncompressed format such as WAV. In order for the web service to return smaller audio formats such as MP3, 
edits would have to be made to the web service so that it converts the byte array into the desired format. This might be done using libraries such as LAME, Tritonus, or possibly even the built-in Java Sound API.
Just to note, the byte array returned by Festival is a full audio file, headers included. Keep this in mind if you wish to conver formats.

###Installation Instructions for Festival
####Requirements
+ Unix-based environment, preferably Linux (Do **NOT** use Windows with Cygwin)
+ A C++ compiler
+ GNU Make any recent version

####Deciding which to version to use
+ Default voices for Festival are generally low quality, so additional voices must be added. There are a higher-quality experimental multisyn voices found among the 1.96 release.
+ Among the most popular additional voices for Festival are the Nitech HTS voices and the enhanced CMU Arctic voices
+ Nitech HTS voices only work on version 1.96 of Festival, which has a harder installation process. There is, however, a single compiled HTS (female) voice available for the 2.1 release; it is found on the festival site with the other 2.1 packages
+ CMU Arctic voices and the experimental multisyn voices work on both the 1.96 and newest 2.1 versions of Festival

####Instructions for installing v1.96
1) Download "festival.tar.gz" and "speech-tools.tar.gz" from the "Festival-1.96-revisions" folder

2) Move these archives into whatever folder you wish Festival to rest in
```
tar xzf festival.tar.gz
tar xzf speech-tools.tar.gz
sudo apt-get install libncurses5-dev
cd speech-tools
./configure
make
make test
make install
cd ../festival
./configure
make
make test (optional)
make install
```

####Instructions for installing v2.1
1) If on Ubuntu, the easiest method is to run "sudo apt-get install festival" to get the Debian version of festival. Otherwise, continue reading for how to install festival from source

2) Go to [http://festvox.org/packed/festival/2.1/] (http://festvox.org/packed/festival/2.1/) and download 
	festival-2.1-release.tar.gz, festlex_CMU.tar.gz, festlex_OALD.tar.gz, festlex_POSLEX.tar.gz, and speech_tools-2.1-release.tar.gz. 
	The other archives are optional, and merely offer a set of working voices. If you wish to test the festival installation using the 
	given test suite, download 	the other the remaining files as well.

3) Unzip all the archives using a "tar xzf <filename>" command and move the "festival" and "speech-tools" folders into whatever directory you wish Festival to rest in

4) Change directories to wherever festival is
```
sudo apt-get install libncurses5-dev
cd speech-tools
./configure
make
make test
make install
cd ../festival
./configure
make
make test
make install
```

####Installing Additional Voices
+ Just follow the instructions [here](http://ubuntuforums.org/showthread.php?t=751169).
+ The experimenal multisyn [here] (http://www.festvox.org/packed/festival/1.96/). Though released with the 1.96 version, they also work with v2.1. The multisyn voices should be moved into a new folder: festival>lib>voices-multisyn folder.
+ Highly recommended are the Enhanced Arctic CMU voices and/or the Nitech HTS voices. CMU voices are far bigger, but work on v2.1, while Nitech only works on v1.96. 
+ For 2.1 users, a packaged female voice (cmu_us_slt_arctic_hts) that is similar to its nitech equivalent may be found on the [festival site] (http://www.festvox.org/packed/festival/2.1/). There are also two additional cg voices on that site (all should be moved to the festival>lib>voices>us folder).
+ After installing the voices, be sure to edit the voices.scm file (located in festival>lib) to include the names of whatever voices you've added. This edit should be done in the "(defvar default-voice-priority-list " function, located around line 325.
+ To make a voice the default voice, either edit the voice list voices.scm so that the desired voice is first on the list, or edit the festival.scm file (located in festival>lib) so that "(set! voice_default 'voice_{NAME OF VOICE}')" is added at the end.
+ **NOTE** The Enhanced Arctic CMU voices CANNOT be set as defaults for some reason. Therefore, to make sure festival is able to run, make sure to have an installed and useable voice (the default diphone voices, or even the Nitech voices) at the top of the voice list. Do not edit the festival.scm file to change the default voice.
+ Demos of the CMU and Nitech voices can be found in the Voice_demos folder. The name of the file is the *full* name of the voice, so this is the parameter that should be used in the web service if any voice selections are chosen.

####Running the Festival server
+ To run the festival server, either export the PATH to point to the bin inside the festival folder (`export PATH={path to main festival folder}/festival/bin:$PATH`), 
	or manually type out the path and do a `festival --server` from the command line. Of course, include the "./" prefix if done without exporting the PATH.
+ By default, Festival runs on port 1314. To change the port at runtime, add "'(set! server_port PORT)'" to the command, where PORT is the desired port number. Ex: `festival --server '(set! server_port 1515)'`
+ To edit any defaults, including the port numbers, number of concurrent clients, security settings, etc...Edit the festival.scm file (located at festival>lib). Information about the server settings begins at around line 408.
+ An init script can be used to force the server to start automatically. An example script for CentOS has been provided. Ubuntu users may refer to [https://github.com/zeehio/festival-debian/blob/master/debian/festival.init] (https://github.com/zeehio/festival-debian/blob/master/debian/festival.init).

####Additional information
+ The Festival manual is located at [http://www.cstr.ed.ac.uk/projects/festival/manual/festival_toc.html] (http://www.cstr.ed.ac.uk/projects/festival/manual/festival_toc.html). Reference it for more details and client/server API.

###Creating the web service
+ An example web service is included in the "festival_service" folder and uses the Jersey framework. It can be compiled using Maven:
```
mvn clean install
```

This will generate a .war file. If you don't wish to change the code, then using the provided .war file is fine.

Note: Some of the required repositories have moved, leaving 301 redirect requests. Maven 2.2.1 fails to follow these requests, leading to build errors. This can be bypassed by either downgrading to 2.2.0 or upgrading to 3.x.


+ Details about the source code:
	- Client.java is the actual Festival client. It communicates with the Festival server using Scheme commands and returns a byte array of the audio file in some uncompressed format (wav, nist, ulaw, etc.). 
		This class handles customization of server location, voice selection, text mode (not demonstrated...see manual), and file type. Currently, the default settings for audio are WAV format and the cmu_us_awb_arctic_clunits voice. Overloaded functions have been provided to change those parameters. 
	- Festival_service.java demonstrates how to call the Festival client. The first function, returnSound() gets the desired text from the URL and returns the audio file in raw, binary format. The second function, returnSound2() receives a JSON and returns a JSON as well. 
		This second function offers an example of how to parse the wav-format header for information about the audio (sampling rate, number of samples). The audio file itself is returned as a base64 string inside the JSON.
		If any audio conversions are to be done, this should be where to do it.
	- Basic_Test.java has nothing to do with the Festival service and is merely an experiment in using the Jersey framework. It can be ignored.





