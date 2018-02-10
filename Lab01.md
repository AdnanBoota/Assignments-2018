# Lab session #1: Basic "Knowledge Toolbox" to get started in the Cloud
In this Lab session, you will be asked to put into practice the basic knowledge required for the Lab sessions of this course.


#  Pre-lab homework 0
Take a look at the following hands-on guides to check if you already have the basic knowledge to follow this course. If not, please do the assignments.
* Hands-on 0: [Run a Linux OS in a Virtual Machine](../../../Cloud-Computing-QuickStart/blob/master/LinuxOS-VirtualMachine.md) (Only required for Windows users)
* Hands-on 1: [Git and GitHub Quick Start](../../../Cloud-Computing-QuickStart/blob/master/Git-Github-Quick-Start.md)
* Hands-on 2: [Markdown syntax](../../../Cloud-Computing-QuickStart/blob/master/Quick-Start-Markdown.md)
* Hands-on 3: [Python Quick Start](../../../Cloud-Computing-QuickStart/blob/master/Python-Quick-Start.md)

#  Pre-lab homework 1
Create an AWS account and install AWS CLI following the following hands-on:
* Hands-on 5: [Getting Started in the Cloud with AWS](../../../Cloud-Computing-QuickStart/blob/master/Quick-Start-AWS.md)

#  Tasks for Lab session #1
## Task 1.1:
Install Python on your laptop.

[PyCharm](https://www.jetbrains.com/pycharm/) is a very popular [IDE](https://en.wikipedia.org/wiki/Integrated_development_environment) that will make your life much easier. It supports execution and debugging of Python, Python environments, code version control, it has a built-in terminal and all kinds of plugins. Moreover, it is [completely free for students](https://www.jetbrains.com/buy/classroom/?product=pycharm).

## Task 1.2:
Create a python code that uses the “random” library. We will positively value if you build a creative program in python. The minimum example accepted will be a code that generates a random number between 1 and 20. Then let the player guess the number introduced, displaying if the number is too low or high. The game ends either when the number is guessed correctly. The suggested program name is `Lab1.guessnumber.py`.
## Task 1.3:
Create a `private repository` **CLOUD-COMPUTING-CLASS-2018** in your GitHub account. Use your student email account (".upc.edu") to create your GitHub account to benefit from private repositories and other perks of the [student pack](https://education.github.com/pack).

We advice you to create a global folder with a subfolder for each lab session similarly as shown below:

<p align="center"><img src="./images/SolutionsFolder.png " alt="Lab01-1" title="Solutions Folder"/></p>

## Task 1.4:   
Update your remote repository from the local repository on your laptop:
```
echo "# CLOUD-COMPUTING-CLASS-2018" >> README.md
git init
git add README.md
git add Lab1/Lab1.guessnumber.py
git commit -m "first commit"
git remote add origin https://github.com/<username>/CLOUD-COMPUTING-CLASS-2018.git
git push -u origin master
```
> change `<username>` for your Github account

It is better that you manage git by hand. Once you become familiar with git you can use PyCharm to save you some typing.

## Task 1.5:
Update the `README.md` file including all the information about your group (member's name and email addresses).
## Task 1.6:
Invite `angeltoribio-UPC-BCN` to your remote private repository as a collaborator using `settings` button (for evaluation purpose).
## Task 1.7:
Create an EC2 instance at AWS. Login and pull down all the contents of your GitHub repository to make an exact clone by using `git clone` command.
## Task 1.8:
Execute the program `Lab1.guessnumber.py` in your AWS instance. Take a screenshot of the terminal window that are you using as proof.
Include that screenshot in your local repository, on your laptop, with the name `Lab1.AWSterminal.png`.
## Task 1.9:
Update your remote GitHub repository with the updated `README.md`and the new file `Lab1.AWSterminal.png` using the `git`commands `add`, `commit` and `push`.
## Task 1.10:
Create an S3 bucket and synchronize your repository there. Take a screenshot of the browser window showing your S3 bucket. Include that screenshot in your local repository, on your laptop, with the name `Lab1.S3Bucket.png`.

# How to submit this assignment:
Submit **before the deadline** to the *RACO Practicals section* a "Lab1.txt" file including:

1. Group number
2. Name and email of the members of this group
3. GitHub URL that contains your lab answers
4. Add any comment that you consider necessary