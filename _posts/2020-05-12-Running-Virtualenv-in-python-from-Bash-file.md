![alt text](/docs/assets/0_IGLjFlKFg4Eozmw8.png)

---

Running Virtualenv in python from Bash file
Even after many searches on google i was not able to find any straight forward solution to this problem. Hence sharing my experience after implementing it. It is very easy and may be many of you would know much part of it.

In my early days of coding I got around a situation where i was required to run many commands in virtual environment in my python project. So it was to write a bash script so that other developers would not have to go through all long process. So one thing where i was struggling was to create virtual environment from bash file…..run everything…..delete virtualenv and then close bash file.

---

**Prerequisutes** :
So lets dive in to solve it. First some basic terms:
- Familiarity with /dev/null or >dev/null 2>&1 . /dev/nullis a file called null device in unix systems. It is called blackhole because whatever redirected to it is discarded and it returns EOF if read. Above 2 stands for stderr and 1 stands for stdout . 2>&1 means redirecting output from std error to std output. For more details there is a good article :

Step by step breakdown of /dev/null
Newcomers to Bash programming will sooner or later come across /dev/null and another obscure jargon: > /dev/null 2>&1…medium.com
The $? symbol in linux is a variable that always contains the exit status of the previous command you ran. An exit code of 0 indicates that the previous command was successful while anything else indicates an error code for that specific program.

colossus@nvc-dgx1-001:~$ cat tmp.txt
hello world
colossus@nvc-dgx1-001:~$ echo $?
0
colossus@nvc-dgx1-001:~$ cat tt.txt
cat: tt.txt: No such file or directory
colossus@nvc-dgx1-001:~$ echo $?
1
colossus@nvc-dgx1-001:~$
As you can see above tmp.txt file is present , if we run cat tmp.txt it is successful with exit status 0 , if file is not present like tt.txt , then $? returns 1. So every command in unix have exit status with some error message (if not successful) . This error message is directed to stderr not stdout. For more reference refer to link mentioned in first bullet point.
type command in linux . It is used to find out the information about a Linux command. As the name implies, you can easily find whether the given command is an alias, shell built-in, file, function, or keyword using "type" command. Additionally, you can find the actual path of the command too. Why would anyone need to find the command type? For instance, if you happen to work on a shared computer often, some guys may intentionally or accidentally create an alias to a particular Linux command to perform an unwanted operation, for example. alias ls = rm -rf / So, it is always good idea to inspect them before something worse happen. This is where the type command comes in help. For more details refer to :

The Type Command Tutorial With Examples For Beginners - OSTechNix
The Type command is used to find out the information about a Linux command. As the name implies, you can easily find…www.ostechnix.com
$ type ls
ls is aliased to `ls --color=auto'
&& in shell command. && lets you do something based on whether the previous command completed successfully - that's why you tend to see it chained as do_something && do_something_else_that_depended_on_something.

Please read these links , it will just take 2 mins but they are worth reading.
What is the purpose of "&&" in a shell command?
command-line - what is the purpose of &&? In shell, when you see $ command one && command two the intent is to execute…stackoverflow.com
Confusing use of && and || operators
Unix & Linux Stack Exchange is a question and answer site for users of Linux, FreeBSD and other Un*x-like operating…unix.stackexchange.com
Linux Bash Syntax: Meaning of &&, \, and -
Is the logical AND: && is a way of expressing Logical AND, meaning the entire expression is True only if both sides of…serverfault.com

---

Code :
Steps to be implemented :
Check if virtualenv is present or not first ? You can check using type command in unix.

```bash
### If present
colossus@nvc-dgx1-001:~$ type virtualenv
virtualenv is /usr/bin/virtualenv
colossus@nvc-dgx1-001:~$ echo $?
0
### If not present
colossus@nvc-dgx1-001:~$ type virtualenv
-bash: type: virtualenv: not found
colossus@nvc-dgx1-001:~$ echo $?
1
virtualenv my_env -p python3 to create virtual environment my_package_env with python3.
source ./my_env/bin/activate to activate virtual environment.
my_env/bin/pip3 install -r ./requirements.txt If there is some requirements (like allure,pytest,json etc) to be installed present in requirements.txt.
my_env/bin/pip3 install dist/my_package-*.whl to install any .whl file of project my_package .
After installing .whl my_pakage will get a binary in my_env/bin/folder.
To run my_package in virtual environment , run my_env/bin/my_package .
deactivate to deactivate the virtual environment.
sudo rm -rf my_env to delete virtual environment my_env .

#!/bin/bash

#TODO based on machine install virtualenv.
#We are redirecting stderr to null file,
#we are just interested in exit status,
#you can also run ! type "virtualenv" > /dev/null
#separately and then capture $? of previous command 
#compare it to 0
# https://stackoverflow.com/questions/26675681/how-to-check-the-exit-status-using-an-if-statement
if ! type "virtualenv" > /dev/null; then
       echo "virtualenv not found"
       exit 1
fi

virtualenv my_env -p python3
source ./my_env/bin/activate && my_env/bin/pip3 install -r ./requirements.txt && my_env/bin/pip3 install dist/my_package-*.whl && my_env/bin/my_package && deactivate

sudo rm -rf my_env
Above code is very easy if you read all bullet points carefully.
Now if you are doing this in git then it is better to create TAG variable with command
TAG=`git describe --dirty`
echo $TAG
virtualenv my_env_$TAG -p python3 
Using this will help you know different virtual environments created with different tags in git of your development branch.

---
```

Thanks for reading . Please click on clap if you like this article.
