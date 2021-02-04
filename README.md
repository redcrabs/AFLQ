# AFLQL - Comperehensive static analysis for guiding fuzzers with dictionaries

AFLQL aims at increasing code coverage and finding more bugs with extensive static analysis to guide the fuzzer through more detailed analysis
	by a set of codeql scripts (Although there are just 2 scripts now, will complete step by step)

AFLQL is basically using CodeQL language to run static analysis on source code level in Open-Source projects to generate dictionaries per codeql 
output files.

This project is built mainly on top of AFL++LTO configuration and we highly recommend to use the optimal configs of AFL++ with LTO and CMPLOG instrumentation
to get most of your fuzzing campaigns . 

It just as simple as passing -x to afl-fuzz and the output directory generated by AFLQL python files to run AFL++ around thousands of dictionaries.

Usage summary :

To use AFLQL following steps are recommended :
1) AFLQL is built on top of AFL++ 2.68 release version, so before every thing run : 
	- `chmod +x aflplusplus-lto-reqs.sh`
	- `sudo su`
	- `./aflplusplus-lto-reqs.sh`

	This will basically will fetch AFL++ for you .

	Now build AFL++ by running at AFL++ Directory (Before this, exit from superuser):
	- `make distrib`
	- `sudo make install`

2) Run fetch-build-ql.sh to fetch, install and config codeql for you, this bash script will:
	- fetch codeql binaries for you
	- clone the necesarry codeql files 
	- setup to run codeql in the correct way
	
	To ensure that codeql is working fine you can run and check the output of :
		- `codeql resolve languages`
		- `codeql resolve qlpacks`
		

3) To generate a project and understand AFLQL and CodeQL, this will generate a project for you automatically so you can run codeql scripts for analysis

4) There is a folder in this directory called "QL-Scripts", go over and run:

	- codeql query run CODQL-QUERY.ql -d DATABASENAME 
		- for example : codeql query run literal.ql -d sample1Db
		- Correct way for ALQL : `codeql query run literal.ql -d sample1Db > ooo-const.txt`
			
5) Use python scripts for analysing the results of codeql scripts .

	- Here we provide 2 python scripts which will generate corpus for you :
		- analysis-constants.py -> for running to analyse the output of "literal.ql"
			- literal.ql will basically extract all constants related to a project, this will help to find some codes, magic codes in a more fast way, so saving the time for fuzzer or your concolic executor (If you possibly use it !)
						
	- analysis-strings.py -> This file is responsible to do analysis for output of "stirngs.ql" 	
		- strings.ql -> This will extract all strings for a given project using extensive static analysis. for big Open-Source projects, this
						will help you alot!

	Example-1 : 
		at first we run : `codeql query run literal.ql -d sample1Db > out-const.txt`
	Example-2 :
		Then we run : `codeql query run strings.ql -d sample1Db > out-str.txt`
				
	Then :
		Analysis-1 :
			- `python2 analysis-constants.py myCorpusDir out-const.txt`  (If you get error after running analysis-constants.py then run "analysis-constants2.py"
					instead
					- This will make myCorpusDir and generate dictionaries for constants in the project
		Analysis-2 :
			- `python2 analysis-strings.py myCorpusDir out-str.txt`
					- This will generate corpus for all strings in the project
					
6) Running the fuzzing campaign :

	- Just make AFL++LTO config as optimal as possible the run your fuzzing campaign as :
		`afl-fuzz -i inputfolder -x myCorpusDir -o outputfolder -m none -c [cmplog-version-program] -- [LTO-Instrumented-program]`
								

---- Making Open-Source projects with CodeQL ----

- Simple enough, do every step till the make stage, for example, consider LAVA-M :
			- `./configure` (As always!) (No instrumentation is needed)
			- `codeql database create whodb --language=cpp --command=make`
				- This is the most important part of codeql for Open-Source and Big Projects
				- Now you should have a database named "whodb" as a folder, now you can run analysis against it .
				
	- Important thing about running codeql scripts :
		- Beginning with running codeql script, you may receive error, it's very possibly that you forgot to create "qlpack.yml" file, please look at
			"generate-demo.sh" file to see how it will be create .
		- You may have "qlpack.yml" in the correct way, but didn't upgrade your datebase, after creating "qlpack.yml" you should upgrade your database as:
			- `codeql database upgrade MYDATABASE`
				This will succesfully upgrade your database, now you should not have problems for running codeql scripts
						
# Technical documentation					
