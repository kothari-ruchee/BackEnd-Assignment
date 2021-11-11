Instructions to run application:

# To build docker containers for both application server and database
docker-compose up --build -d 



# Migrate data into database (database name:test, user:test, password : password)
docker exec <server_container_name> npm run migrate


# Seed data into database
docker exec <server_container_name> npm run seed


# To delete the docker containers 
docker-compose down
