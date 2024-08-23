# Docker Compose Setup for a Python Flask API with Nginx Reverse Proxy

> In this tutorial, we will explore a comprehensive setup using `Docker Compose` for a multi-container application. The scenario involves a Python Flask `backend API`, a simple `front-end interface`, and an `Nginx` reverse proxy server. This setup is designed to run seamlessly on both local development environments and production servers, such as `AWS Linux 2`. The configuration management is achieved through environment files, making it easy to customize settings for different deployment scenarios.

---

## Prerequisites
- Docker installed on your machine or server
- Docker Compose installed on your machine or server

## What is Docker?
<img src="https://upload.wikimedia.org/wikipedia/en/f/f4/Docker_logo.svg" alt="Flask" height="50"/>

Docker is a tool that helps developers package and run their applications smoothly, ensuring they work the same way on any computer. It simplifies managing and sharing software, making it easier to build and deploy applications.

## What is Docker Compose?
<img src="https://raw.githubusercontent.com/docker/compose/main/logo.png" alt="Docker Compose Logo" height="100"/>

Docker Compose is a tool for running multi-container applications on Docker defined using the Compose file format. A Compose file is used to define how one or more containers that make up your application are configured.

---

1. Project Structure:
   ```plaintext
   Docker-Compose-Tutorial/
   │
   ├── back-end/
   │    └── .env
   ├── front-end/
   │    └── index.html
   ├── nginx/
   │    └── nginx.conf
   └── docker-compose.yml
   ```

2. Backend Setup:
   
   - Refer to the [Dockerizing a Python Flask App: A Step-by-Step Guide to Containerizing Your Web Application](https://medium.com/@geeekfa/dockerizing-a-python-flask-app-a-step-by-step-guide-to-containerizing-your-web-application-d0f123159ba2) tutorial to build a Docker image.

   - If you follow step-by-step tutorial, you will be able to Build a Docker Image.
      ```bash
      sudo docker build -t api-flask .
      ```
  
   - Add the following content to `back-end/.env`
      ```
      DOMAIN=localhost
      PORT=80
      PREFIX=/api
      ```
   

3. Frontend Setup:
   - Add the following content to `front-end/index.html`
      ```html
      <!DOCTYPE html>
      <html lang="en">
      <head>
         <meta charset="UTF-8">
         <meta name="viewport" content="width=device-width, initial-scale=1.0">
         <title>API Request Example</title>
      </head>
      <body>
         <h1>Book List</h1>
         <ul id="bookList"></ul>

         <script>
            document.addEventListener("DOMContentLoaded", function () {
                  // Function to make the API request
                  function fetchBooks() {
                     fetch('http://localhost/api/books')
                        .then(response => response.json())
                        .then(data => displayBooks(data))
                        .catch(error => console.error('Error fetching books:', error));
                  }

                  // Function to display the retrieved books
                  function displayBooks(books) {
                     const bookList = document.getElementById('bookList');

                     books.forEach(book => {
                        const listItem = document.createElement('li');
                        listItem.textContent = `${book.title} (ID: ${book.id})`;
                        bookList.appendChild(listItem);
                     });
                  }

                  // Call the fetchBooks function when the page is loaded
                  fetchBooks();
            });
         </script>
      </body>
      </html>
      ```
   - In this example, we use a simple HTML to call `/api/books` API. Keep in mind that for more advanced and dynamic interactions, you would typically use JavaScript frameworks like `Vuejs`, but for the purpose of simplicity, this HTML form should suit.
  
4. Nginx Reverse Proxy:
   - Add the following content to `nginx/nginx.conf`
      ```sh
      server {
        listen       80;
        server_name  localhost;
        root /var/www/front-end;

        location /api {
                proxy_pass http://back-end:5000;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
        }
      }
      ```
      - `listen 80;` Specifies that the server will listen on port 80 for incoming connections.
      - `server_name localhost;` Sets the server name to ***localhost*** This is the domain name that the server considers itself responsible for handling.
      - `root /var/www/front-end;` Defines the root directory for this server block. This is the base directory from which Nginx will look for files to serve.
      - `location /api` Begins a location block for requests matching the ***/api*** URI prefix.
      - `proxy_pass http://back-end:5000;` Forwards requests to the ***/api*** URI to the backend server at ***http://back-end:5000***. This is a reverse proxy configuration where Nginx acts as an intermediary between the client and the backend server.
      - `proxy_set_header lines` These lines set various HTTP headers before forwarding the request to the backend server. They are used to convey information about the client's request to the backend server.

5. Docker Compose Configuration:
   > This Docker Compose file is a basic configuration for running an Nginx container serving a static front-end and another container running a Flask API. The two containers can communicate, with Nginx acting as a reverse proxy for the Flask API.

   - Add the following content to `docker-compose.yml`
      ```yaml
      version: '3'

      services:
         nginx:
            image: nginx
            volumes:
               - ./nginx:/etc/nginx/conf.d
               - ./front-end:/var/www/front-end
            ports:
               - "80:80"

         back-end:
            image: api-flask
            volumes:
               - ./back-end/.env:/api-flask/.env
            expose:
               - "5000"
      ```
      - `version: '3'` Specifies the version of the Docker Compose file format. In this case, it's version 3.
      - `services:` Begins the section where individual services are defined.
      - `nginx:` Defines the ***nginx*** service.
      - `image: nginx` Specifies the Docker image to use for the nginx service, in this case, the official Nginx image from Docker Hub.
      - `volumes:` Defines volumes to be mounted for the nginx service.
      - `- ./nginx:/etc/nginx/conf.d` Mounts the local ./nginx directory into the container's /etc/nginx/conf.d directory. This allows you to provide custom Nginx configuration files.
      - `- ./front-end:/var/www/front-end` Mounts the local ./front-end directory into the container's /var/www/front-end directory. This is likely where the static front-end files are stored.
      - `ports:` Specifies port mappings for the nginx service.
      - `- "80:80"` Maps port 80 on the host to port 80 on the nginx container. This allows you to access the Nginx web server from the host machine.
      - `back-end:` Defines the back-end service.
      - `image: api-flask` Specifies the Docker image to use for the ***back-end*** service, presumably the custom image named ***api-flask*** that you have built in ***Step 2***.
      - `volumes:` Defines volumes to be mounted for the back-end service.
      - `expose:` Exposes ports without publishing them to the host machine.
      - `- "5000"` Exposes port 5000 on the back-end service. This is the port where the backend server is presumably listening for incoming requests.

6. Docker Compose Up:
      - Run the following command to bring up the entire stack:
         ```bash
         sudo docker-compose up -d
         ```
7. Testing:
      - Open your browser and navigate to http://localhost. You should see the the Book List on the page.
      - Also by navigating to http://localhost/api, You should see the Swagger page and be able to interact with the APIs.



---

# Docker on AWS Linux 2

1. Make sure you have pushed [Api-Flask](https://github.com/geeekfa/Api-Flask) Docker image to Docker Hub. If you don't have this Docker image in your machine, read [Dockerizing a Python Flask App: A Step-by-Step Guide to Containerizing Your Web Application](https://medium.com/@geeekfa/dockerizing-a-python-flask-app-a-step-by-step-guide-to-containerizing-your-web-application-d0f123159ba2) article. By the way, run the following command to push the Docker image to Docker hub.
   ```bash
   docker login

   # amd64
   docker buildx build --platform linux/amd64 -t <your_docker_hub_username>/api-flask:latest-amd64 --push .

   # arm64
   docker buildx build --platform linux/arm64 -t <your_docker_hub_username>/api-flask:latest-arm64 --push .
   ```

2. Launch an AWS Linux 2 Instance:
   - Start by launching an AWS EC2 instance with the Amazon Linux 2 AMI.

3. Open a terminal and `SSH` to the AWS EC2 instance:
      ```bash
      ssh -i <YOUR_PRIVATE_KEY.pem> <YOUR_USER_NAME>@<THE_AWS_EC2_PUBLIC_DNS_ADDRESS>
      ```

4. Update the System:
   ```bash
   sudo yum update -y
   ```

5. Install Docker and enable Docker to start on boot:
    ```bash
    sudo yum install -y amazon-linux-extras
    sudo amazon-linux-extras install docker
    sudo systemctl start docker
    sudo usermod -a -G docker $USER
    sudo systemctl enable docker
    ```

6. Verify Docker Installation:
   - Close and reopen the terminal to run the following command.
        ```bash
        docker info
        ```

7. Install the last version of Docker Compose:
    ```bash
    sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    ```

8. Verify Docker Compose Installation:
   ```
   docker-compose version
   ```

9.  Some modifications:
      - Clone [Docker-Compose-Tutorial](https://github.com/geeekfa/Docker-Compose-Tutorial) repository If you don't have it in your machine.
      - Open `docker-compose.yml`
        - Modify 
            ```yaml
            back-end:
               image: api-flask
            ```
            To
            ```yaml
            back-end:
               image: <your_docker_hub_username>/api-flask:latest-amd64
            ```
            This modification is because of that `docker-compose` on AWS EC2 instance must pull the `api-flask` Docker image from Docker Hub at first. finally, after the modification, your `docker-compose.yml` is like the following:
            ```yaml
            version: '3'

            services:
               nginx:
                  image: nginx
                  volumes:
                     - ./nginx:/etc/nginx/conf.d
                     - ./front-end:/var/www/front-end
                  ports:
                     - "80:80"

               back-end:
                  image: <your_docker_hub_username>/api-flask:latest-amd64
                  volumes:
                     - ./back-end/.env:/api-flask/.env
                  expose:
                     - "5000"
            ```
       - Open `nginx/nginx.conf`
          - Modify
            ```html
            server_name  localhost;
            ```
            To
            ```html
            server_name  <THE_AWS_EC2_PUBLIC_DNS_ADDRESS>;
            ```
            finally, after the modification, your `nginx/nginx.conf` is like the following:
            ```html
            server {
               listen       80;
               server_name  <THE_AWS_EC2_PUBLIC_DNS_ADDRESS>;
               root /var/www/front-end;

               location /api {
                        proxy_pass http://back-end:5000;
                        proxy_set_header Host $host;
                        proxy_set_header X-Real-IP $remote_addr;
                        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                        proxy_set_header X-Forwarded-Proto $scheme;
               }
            }
            ```


      - Open `.env`
          - Modify
            ```
            DOMAIN=localhost
            PORT=80
            PREFIX=/api
            ```
            To
            ```html
            DOMAIN=<THE_AWS_EC2_PUBLIC_DNS_ADDRESS>
            PORT=80
            PREFIX=/api
            ```
      - Open `front-end/index.html`
          - Search `localhost` and replace with `<THE_AWS_EC2_PUBLIC_DNS_ADDRESS>`
  
10. Copy Docker Compose content to the AWS EC2 instance:
    - Open a new terminal and `cd` to the `Docker-Compose-Tutorial` folder.
    - Copy the folder to the AWS EC2 instance.
      ```bash
      scp -i <YOUR_PRIVATE_KEY.pem>  -r . <YOUR_USER_NAME>@<THE_AWS_EC2_PUBLIC_DNS_ADDRESS>:~/Docker-Compose-Tutorial

      ```

11. Return to the terminal from which you `SSH` to the AWS EC2 instance.
      ```bash
      cd Docker-Compose-Tutorial
      docker-compose up -d
      ```

12. Testing:
    - Open your browser and navigate to [http://<THE_AWS_EC2_PUBLIC_DNS_ADDRESS>](). You should see the the Book List on the page.
    - Also by navigating to [http://<THE_AWS_EC2_PUBLIC_DNS_ADDRESS>/api](), You should see the Swagger page and be able to interact with the APIs.