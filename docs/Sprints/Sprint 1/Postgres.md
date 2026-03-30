# Postgres Installation

Once time we have the package of docker. we execute the nex command in our terminal

```bash

docker run --name name-proyect -e POSTGRES_PASSWORD=my-password -d postgres

```
_This has making all of necesary to run our image docker of postgres_
**With this you alredy to use you service of postgres on docker**

##### List of services running in you machine
```bash

    docker ps
```
##### Run the service of postgress
```bash

    docker run <name-image>
```

##### Stop the service of postgress
```bash

    docker stop <name-image>
```

##### Start the service of postgress
```bash

    docker start <name-image>
```

##### Delete the service of postgress
```bash

    docker rm <name-image> or <identifier-image>
```