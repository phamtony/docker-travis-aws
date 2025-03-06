docker build -f Dockerfile.dev .

#wont work
docker run <image id>

#will work because of porting
docker run -p 3000:3000 <image id>

docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <image id>
# no need to use $(pwd), use pwd (oh my zsh)


-v is volume
pwd is path to folder
: is indication of reference, having container reference the local folder after the : (which is /app)


-v /app/node_modules 
doesn't reference, bypasses that and has it refer to what's in the container


#Docker compose:

docker compose yml 

    ports: 
      -"3000:3000"
    volumes:
      - /app/node_modules
      - .:/app


For testing:
without compose: docker run -it <container_id> npm run test


#Docker compose testing

  web: 
    build: 
      context: .
      dockerfile: Dockerfile.dev
    ports: 
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
  test:
    build: 
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - /app/node_modules
      - .:/app
    command: ["npm", "run", "test"]  



 Dockerfile for production, using nginx

build phase and run phase 2 diff images for one dockerfile









