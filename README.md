# RandomCTFdNotes

This README exists to give information on how to set up the [CTFd](https://ctfd.io/) platform to run in tandem with the [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/). All information contained within has been gleaned from the [awesome documentation](https://bkimminich.gitbooks.io/pwning-owasp-juice-shop/content/part1/ctf.html) on how to run your own Juice Shop CTF. The only real difference is that this guide details specific instructions step by step using docker containers for both CTFd and Juice Shop and the optimal order to do all the things.
The whole process is a little finicky on first run through, but after the first run it's very easily repeatable.


# Prerequisites
- [NPM](https://www.npmjs.com/get-npm)
- [Juice Shop CTF CLI](https://www.npmjs.com/package/juice-shop-ctf-cli)
- [Docker](https://docs.docker.com/get-docker/)
- [Juice Shop](https://hub.docker.com/r/bkimminich/juice-shop) and [CTFd](https://hub.docker.com/r/ctfd/ctfd) Docker containers downloaded but not running

# Getting Started
## Before Generating a CTFd Config File
Before generating a reusable config file you will have to decide two things, what will your secret key be (it's called a secret key but it doesn't have to be kept super secret like a password, just think of it as a string that people will need to use during this setup so that everything works together) and what type of CTF hints you want.
**Note The secret key is used so that anyone can run Juice Shop with the key as an environment variable, allowing them to integrate with the CTFd platform. If a user wishes to run Juice Shop from source, Heroku, AWS or other environment all they need to do is set this variable to function correctly with the CTFd platform that uses the same key**

There are three types of hints to choose from;
- None
- Text
- URL
None is fairly self explanatory, text hints provide a bit of information on each Juice Shop challenge and URL hints link to a web page that give further information on the challenge, what the challenge is trying to do and how you might think or go about something similar. The URLs will not provide answers.
Along with these types of hints, you've got another choice to make, free hints, or paid hints. Free hints are, of course, free. Paid hints will cost the user points to unlock, so a user must spend their points earned on other challenges to view the hints for another challenge.
If in doubt, a safe bet is free text and URL hints. You can always run through the setup ad give it a test, it's very easy to change the configuration afterwards.

### Temporarily Running the Juice Shop Container
To ensure that the CTF config file generated is 100% accurate, the Juice Shop docker container should be running locally. This is not entirely necessary but the reason for this is that when generating a config file the CTFd CLI asks for a URL to grab the challenge information from. The default URL provided in the CLI can be used, however that will point to the Heroku instance of Juice Shop. Why is this a bad thing, you may ask, well there are a small number of challenges that are not possible within the Heroku environment, this will mean that using the default URL will cause some challenges to have a note 'this is not possible on Heroku' which is not accurate for us when running docker instances. Similarly, there are a number of challenges that are not possible within Docker, so we want to point our CLI at a running Juice Shop docker instance to guarantee that the challenges generated match the environment our users will be running the CTF on.

- `docker run -p 3000:3000 bkimminich/juice-shop`
- Navigate to `http://localhost:3000` in a browser to make sure Juice Shop is up and running correctly

## Generating a Config File
Generate a config file using the CLI installed earlier using NPM:
`juice-shop-ctf`
The CLI will then prompt a number of questions. Enter the following:
- CTFd
- `http://localhost:3000`
- Enter your secret key
- Select the type of text hints you want
- Select the type of URL hints
A zip file will be generated with the configuration options we selected.

### Stopping the Juice Shop Container
Our Juice Shop container will need to be run later on with some modified commands so it's safe to exit the running container we started previously.
Go to the terminal tab or window that was running Juice Shop and press `CTRL C` to stop the container.

# Setting up CTFd
Now that we have our config file we can start setting up the CTFd platform. The first part of the setup doesn't matter very much, however the second part is important.

- Spin up the CTFd Docker container in detached mode:
- `docker run -d -p 8000:8000 ctfd/ctfd`
- Navigate to `http://localhost:8000` to view the CTFd platform
- Run through the setup options, the data used here doesn't matter so don't worry about getting things 100% accurate
- Once logged in as an admin navigate to Admin Panel > Config > Backup > Import
- Import the zip file generated earlier by the CLI
- This will then run through the setup once more, now is the important time to get the details correct
- After running through the setup once more, make sure to register all users and update their profiles as admin and update the index page HTML as you wish
- Once all users and modifications have been made, again navigate to the Backup section of the Admin Panel Config
- Export the current platform settings as a zip file
- Now we have a backup zip file that can be used to reset the platform to a blank state if we want to run multiple CTF events!

# Testing the CTFd Platform
To make sure we have everything properly configured to work with Juice Shop, we will want to spin up Juice Shop, complete a challenge and then submit it on CTFd to verify that everything works.

### Startomg the Juice Shop Docker Container
Execute the following command in the terminal to spin up the Juice Shop docker container in detached mode with the correct environment variables.

`docker run -d -e "NODE_ENV=CTF" -e "CTF_KEY=your-secret-key-from-earlier-or-provided-by-the-ctf-host" -p 3000:3000 bkimminich/juice-shop`

Once up and running, navigate to `http://localhost:3000` to view Juice Shop and complete any of the challenges (An easy task to test this would be viewing the Score Board or the Privacy Policy).

## Verifying a Captured Flag
On completion of any Juice Shop challenge a long string will be displayed, this is the 'flag', copy this to the clip board
- Log in to the CTFd platform (as any type of user)
- Navigate to Challenges
- Create a team
- Click on the challenge that you completed
- Submit the flag copied from the Juice Shop instance

If the flag does not work, ensure that the same key was used to generate the CTFd config file via the CLI that was used as the environment variable when starting the Juice Shop Docker Container

### Stopping the Docker Containers
If you decide that you no longer want to run Juice Shop or CTFd locally but are new to this whole Docker thing then execute the following commands to stop the container(s).

- Run `docker ps` in the terminal to view all running docker instances
- Copy the 12 character string under the name column for example `0fbe7526861e` (multiple names can be copied and used at the same time in the following command)
- Run `docker stop 0fbe7526861e` to stop the docker container
- Stop multiple containers at once by separating the names by a space, eg `docker stop 0fbe7526861e 5a987e01b87c`
