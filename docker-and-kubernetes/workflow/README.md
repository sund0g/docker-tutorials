## Section 6: Creating a Production-Grade Workflow

The objectives for this section are,

1. Understand and create a product workflow from developer code check-in to production deployment, (using a React app as the example). The app will be what is dockerized for the workflow, (lessons 56-61).

	The tools used to create the workflow are,
	
	* GitHub
	* TravisCI
	* AWS Elastic Beanstalk

2. Learn how to tell Docker to build with non-standard Dockerfile names (lesson 62),

	e.g. `docker build -f dev.Dockerfile .`

3. Learn to recognize container bloat and control it, (lesson 63). When the React app was created, all the app dependencies were auto-generated in the **node_modules** subdirectory of "frontend". These are not needed in the container because it is built from the **node:alpine** image which already contains these dependencies, therefore, the **node_modules** directory can be deleted. If it was not deleted, the output of `docker build ...` would be something like,

		Sending build context to Docker daemon  238.7MB
		...
		
	Monitoring the build log output will provide clues as to how to keep the container size as small as possible.

	>**It is best practice to always clean up directories before adding them to the container.** This will keep the size of the container from becoming unnecessarily large.

4. Learn about **volumes** and how they can be used to update content without stopping/rebuilding/restarting containers, (lessons 66-67).

5. Learn how to leverage **docker-compose.yml** to make starting and stopping the development environment more efficient, (lessons 68-70).

6. Learn about testing, (lessons 71-73). Also an issue with testing and docker-compose using [docker attach](https://docs.docker.com/engine/reference/commandline/attach/), (lesson 74).

7. Learn how to create a production-ready container using [nginx](https://www.nginx.com/), (lessons 75 and 78).

8. Learn about [multi-stage builds](https://docs.docker.com/v17.09/engine/userguide/eng-image/multistage-build/#use-multi-stage-builds), (lessons 76 and 77).

## Project-specific setup

This lesson requires some additional setup as follows below. **NOTE:** The "frontend" directory is not checked into the repo because its contents will be created in [step 3](#3)

1. Install [Nodejs](https://nodejs.org/) from either the website, (via download), or if on a Mac, via Brew 

		brew install node
	
2. Install [create-react-app](https://www.npmjs.com/package/create-react-app) which will be used to create the React app,

		npm install -g create-react-app
	
	(Additional info on create-react-app is [here](https://reactjs.org/docs/create-a-new-react-app.html))

3. Create the app, (for details on the app, refer to the README.md in the "frontend" directory),

		create-react-app frontend
		
## Building the containerized application

### For a Development environment
Execute,

	docker build -f dev.Dockerfile . -t <username>/frontend

### For a Production environment
Execute,
	
	docker build -f prod.Dockerfile . -t <username>/prod-frontend 
	
## Starting the container

### In a Development environment

This almost goes without saying, *but...*

When starting a container from the app image, with the command `docker run frontend` the log output will have something like, 

	frontend@0.1.0 start /app
	react-scripts start
	
	Starting the development server...
	
	Compiled successfully!
	
	You can now view frontend in the browser.
	
	Local:            http://localhost:3000/
	On Your Network:  http://172.17.0.2:3000/
	
If you enter the local address into a browser window, it will return, "the site cannot be reached".

This is because the **docker run** command did not expose any ports outside of the Docker network.

Remember the Docker CLI has to include the ports, e.g.

	docker run -p 3000:3000 frontend

Additionally, the volumes must also be added to the command, (lessons 66 and 67). The full command now looks like,

	docker run -p 3000:3000 -v /app/node_modules -v $(pwd)/frontend:/app frontend
	
All that being said, **docker-compose.yml** has been created, (lessons 68 and 69) to make the development setup more efficient. Please refer to it for details.

To start the development environment, execute

	docker-compose up
	
### In a production evironment

By the end of Section 6, we have learned how to create a production-ready workflow using nginx as the web server to host the frontend application. To start the server, execute

	docker run -p 8080:80 <username>/prod-frontend
	
The standard user port 8080 is used to forward traffic to the default nginx port 80.

## Stopping the container

In the **development environment** (docker-compose), execute

	docker-compose down
	
In a **production environment**, for now just **`CTRL-C`** to stop the container.




	

	

