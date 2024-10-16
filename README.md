# CI-CD-app
#THIS CONTAINS MY PERSONAL LEARNING DRAFT, YOU MAY FIND SOME WEIRD WORDS

***************************************************************
*******************GITLAB CI/CD DEVOPS*************************
***************************************************************


first of all, you need a gitlab account, so create one

then create a new project --> create a pipeline file : create a new file - name it .gitlab-ci.yml

inside this file, write the jobs/tasks to do, example of what i did : 

stages:
  - build
  - test

build the car :
  stage : build
  script :
    - mkdir build  
    - cd build
    - touch car.txt
    - echo "3aw" > car.txt
  artifacts:
    path : build/

test the car :
  stage : test
  script :
    - test -f build/car.txt
    - cd build
    - grep "3aw car.txt"
    
let's break it down :
	- stages indicates the order of the jobs, in this cas, the stage "build" that is associated to the job "build the car" is the first step
	- then you can see the scripts inside each job having tasks to do
	- and because after a job finishes, the files and directory produced are deleted, we use "artifacts" to save what we want to keep by writing the path of the 
	things we want to keep, in this way, you'll find the generated files/folders under artifacts in build section
	- AND WATCH OUT !!!! ONLY USE TWO SPACES AS IDENTATION AND NOT TABS !!!
	
now we'll try on a project, in order to follow the mentor tutorial, we install node and npm,and also gatsby, idk what the hell is this but i'll follow on

apparently it's a simple library/framework, i installed it created a new simple website (gatsby new my-website), i think i could just do one with angular but it's alright (watch out for
node and gatsby versions comtability)

i created a token, keep a eye on it for me ******************

dude that guy secked ass in last video, i personally downloaded vscode and deployed the project there, then configured my own access token for the project and configured it
git cmd

now once get done with the access thing let's get to creating a pipeline, this time we'll do it from within the project on our IDE and then push it to the gitlab : 

stages:
 - buildMyApp
 - tests

build website:
  stage: buildMyApp
  image: node
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
  artifacts:
   paths:
    - ./public

test website:
  stage: tests
  image: alpine
  script:
   - grep "gatsby" ./public/index.html

test website 2 :
  stage: tests
  image : node
  script:
   - npm install
   - npm install -g gatsby-cli
   - gatsby serve &


let's break it down :

	- stages to coordinate the order of the jobs, here build website wil begin, then we have 2 jobs that'll run in parallel, "test website" and "test website 2"
	- after installing image and running some scripts, we save the ./public folder in a artifact
	- then in the tests, we test the running of the website in one job, and the search of the word "gatsby" in another
	- the image alpine in the test is just to make the job faster, as alpine is a super smaller image of linux so it takes much lesser to build (seconds)
	- the & after the serve is to tell tje pipeline that run this job in background

******************************************************** DEPLOY*****************************
little off topic bu hey, wanna deploy ur app on a cloud rapidly ? 

	- npm install --global surge
	- surge
	- put an email and password
	- put the directory of the project and name the cloud as you want
	- Voilà !
**********************************************************************************************

in case you want to secure variables : gitlab project --> settings --> CI/CD --> Variables then you can disable protect variable and mask it

so in general, every .gitlab-ci.yml has a job atleast, within the job there is a script, stage, it can have an image, artifacts to save, and don't forget the stage section to
determine the order of the execution of the jobs

now now you really think the deploy on surge cloud was for NOTHING ??

we'll try to deploy now throug CI pipelines, we just create a new stage deploy in the .yml file, and then we create the job and put the script for deploying a surge cloud inside it
but in the script, we can't provide the username and password (until now in the tutorial), so we put the login and a token (the output of the "surge token" command in our terminal) in the environment variables in gitlab

updated pipeline code

image: node

stages:
 - buildMyApp
 - tests
 - deploy

build website:
  stage: buildMyApp
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
  artifacts:
   paths:
    - ./public

test website:
  stage: tests
  image: alpine
  script:
   - grep "gatsby" ./public/index.html

test website 2 :
  stage: tests
  script:
   - npm install
   - npm install -g gatsby-cli
   - gatsby serve &

deploy app :
  stage: deploy
  script:
    - npm install --global surge
    - surge --project ./public --domain a77a123.surge.sh

as you can see here, the deploy app stage we just added is for the deployement on a surge cloud, and we can see that we added a image line at the beginning, its for decalring
a general imafe by default that'll be used by other stages

little tour on Gitlab ci fundamentals :

	-you can schedule the pipeline runnings --> Build --> Pipeline schedules --> create new pipeline
	- you know caches ? you can use them if you have multiple jobs  that have the same dependencies installed, for examle, in our yml, we have 2 jobs that 
	installs npm packages, so in this case, cahce would be beneficial (well a little, it's more advantageous in bigger projects), en gros it install in the cache
	the files/libraries.. that are used multiple time, so the job will get the cache file instead of downloading the package, example :

	cache:
	  key: ${CI_COMMIT_REF_SLUG}
	  paths:
	    - node_modules/

and in case some jobs doesn't even need to get these caches just add :  cache: {}

	-environments : to create a pre-prod,prod,psot... just add a environment: name:  url: we'll give an example and then you can see in the gitlab the different environments with buttons leading to their links

well apparently this section is just about some fundamentals notions that aren't that hard, i'll just mention them here and go make your search, they're SOOO ez 
to implement:
	- variables
	- triggers using 'when' inside the jobs + the allow_failure combo to implement manual deployment usage
	
now now here it is important !!! we are configuring merge requests, you know sometimes if a developer creates a branch, and merges it to develop or master,
we don't want to redeploy on the cloud let's say unless it is merging on master (just an example), so we can configure that by using the 'only' and name the branch
like that:

.
.
.
only:
  - master

additional steps : in gitlab, make the option to push in master branch to 'no one', and that the pipelines must succeed and the discussions mut be resolved, like that
ur sure no unwanted pushes are made, and only by full functional merge requests that the code will be pushed to master

	-another aspect, create some dynamic environnments, like the "only" tage use it to publish on merge request to create a environment for developers for example, another for testers... and so on and then destroy finished environments ...

now some YAML Basics:
	- how to disable a job ? ---> put a dot before it (".build : ...") idk why don't just comment it
	- anchor : is the functionality to anchor and relate a value to a field, example:
	
game: &favGame
  name: AC 4
  release date : XX/XX/XXXX
  
person:
  <<: *favGame
  name: &name Kev
  age: 25
  self: *name
  
here for example, we are using the & to tag some values and field, and to associate them to another field, we use the asterisk *, here we tagged the game object with the &favGame, now the person will have the
2 fields declared in the game with the same values, same thing with the name field that we injected its value in self field
