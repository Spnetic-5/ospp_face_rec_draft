# Face Recognition Bootcamp Project using Milvus.

This demo uses [Milvus Vector Database](https://milvus.io/) for storing face embeddings & performing face similaity search based on those stored embeddings. Milvus is a vector similarity search engine, a tool that lets you quickly find the closest matching vector in a pool of billions of vectors.


The system architecture is as below:
<p align="center">
<img src="https://user-images.githubusercontent.com/66636289/191336436-9c37056a-b337-4f65-8f71-373cbc057e02.png" width = "450" height = "600" alt="system_arch" />
</p>

## Data Source

This demo uses the dataset of around 800k images consisting of 1100 Famous Celebrities and an Unknown class to classify unknown faces. All the images have been scraped from Google and contains no duplicate images. Each Celebrity class(folder) consists approximately 700-800 images and the Unknown class consists of 100k images.

> Note: You can also use other images for testing. This system supports the following formats: .jpg and .png.

## Local Deployment

### Requirements

- [Milvus](https://milvus.io/docs/v2.0.0/install_standalone-docker.md)
- [MySQL](https://hub.docker.com/r/mysql/mysql-server)
- [Python3](https://www.python.org/downloads/)
- [Docker](https://docs.docker.com/engine/install/)

## Option 1: Deploy with Docker Compose

The reverse image search system requires Milvus, MySQL, WebServer and WebClient services. We can start these containers with one click through [docker-compose.yaml](./docker-compose.yaml).

- Modify docker-compose.yaml to map your data directory to the docker container of WebServer
```bash
$ git clone https://github.com/milvus-io/bootcamp.git
$ cd solutions/reverse_image_search/quick_deploy
$ vim docker-compose.yaml
```
> Change line 73: `./data:/data` --> `your_data_path:/data`

- Create containers & start servers with docker-compose.yaml
```bash
$ docker-compose up -d
```

Then you will see the that all containers are created.

```bash
[+] Running 12/12
 ⠿ standalone Pulled                                                                                                                                                                          28.5s
   ⠿ 171857c49d0f Pull complete                                                                                                                                                                4.6s
   ⠿ 419640447d26 Pull complete                                                                                                                                                                4.9s
   ⠿ 61e52f862619 Pull complete                                                                                                                                                                5.0s
   ⠿ 2580b47486e5 Pull complete                                                                                                                                                               20.3s
   ⠿ cd742921730d Pull complete                                                                                                                                                               22.2s
   ⠿ 936cb7027fe4 Pull complete                                                                                                                                                               22.2s
   ⠿ 319dd389c04d Pull complete                                                                                                                                                               24.2s
   ⠿ 543c11caaeb6 Pull complete                                                                                                                                                               24.3s
   ⠿ 06d62b89360c Pull complete                                                                                                                                                               24.5s
   ⠿ 5186d5863148 Pull complete                                                                                                                                                               24.6s
   ⠿ b410b80e82c0 Pull complete                                                                                                                                                               24.7s
[+] Running 3/3
 ⠿ Container milvus-minio       Started                                                                                                                                                        1.3s
 ⠿ Container milvus-etcd        Started                                                                                                                                                        1.3s
 ⠿ Container milvus-standalone  Started  
```

And show all containers with `docker ps`, and you can use `docker logs img-search-webserver` to get the logs of **server** container.

```bash
CONTAINER ID   IMAGE                                      COMMAND                  CREATED         STATUS                   PORTS                                           NAMES
3c8d3ceab1ab   milvusdb/milvus:v2.0.2                     "/tini -- milvus run…"   4 minutes ago   Up 4 minutes             0.0.0.0:19530->19530/tcp, :::19530->19530/tcp   milvus-standalone
43c1353b98a2   minio/minio:RELEASE.2020-12-03T00-03-10Z   "/usr/bin/docker-ent…"   4 minutes ago   Up 4 minutes (healthy)   9000/tcp                                        milvus-minio
ab2c1c06ea1e   quay.io/coreos/etcd:v3.5.0                 "etcd -advertise-cli…"   4 minutes ago   Up 4 minutes             2379-2380/tcp                                   milvus-etcd
```


## Option 2: Deploy with source code

> For installing all the required packages & libraries, Please run the following command:
```bash
$ git clone https://github.com/milvus-io/bootcamp.git
$ pip install -m requirements.txt
```

The face recognition bootcamp system requires Milvus, MySQL, WebServer and WebClient services. We can start these containers with one click through [docker-compose.yaml](./docker-compose.yaml).

### 1. Start Milvus & Mysql

First, you need to start Milvus & Mysql servers.

Refer [Milvus Standalone](https://milvus.io/docs/v2.0.0/install_standalone-docker.md) for how to install Milvus. Please note the Milvus version should match pymilvus version in [config.py](./server/src/config.py).

There are several ways to start Mysql. One option is using docker to create a container:
```bash
$ docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d --name qa_mysql mysql:5.7
```


### 2. Start API Server

Then to start the system server, and it provides HTTP backend services.

- **Install the Python packages**

```bash
$ git clone https://github.com/milvus-io/bootcamp.git
$ cd solutions/reverse_image_search/quick_deploy/server
$ pip install -r requirements.txt
```

- **Set configuration**

```bash
$ vim src/config.py
```

Modify the parameters according to your own environment. Here listing some parameters that need to be set, for more information please refer to [config.py](./server/src/config.py).

| **Parameter**    | **Description**                                       | **Default setting** |
| ---------------- | ----------------------------------------------------- | ------------------- |
| MILVUS_HOST      | The IP address of Milvus, you can get it by ifconfig. | 127.0.0.1           |
| MILVUS_PORT      | Port of Milvus.                                       | 19530               |
| VECTOR_DIMENSION | Dimension of the vectors.                             | 1000                |
| MYSQL_HOST       | The IP address of Mysql.                              | 127.0.0.1           |
| MYSQL_PORT       | Port of Mysql.                                        | 3306                |
| DEFAULT_TABLE    | The milvus and mysql default collection name.         | milvus_img_search   |

- **Run the code**

Then start the server with Fastapi.

```bash
$ python src/main.py
```

- **API Docs**

After starting the service, Please visit `127.0.0.1:5000/docs` in your browser to view all the APIs.

![fastapi](pic/fastapi.png)

> /data: get image by path
>
> /progress: get load progress
>
> /img/load: load images into milvus collection
>
> /img/count: count rows in milvus collection
>
> /img/drop: drop milvus collection & corresponding Mysql table
>
> /img/search: search for most similar image emb in milvus collection and get image info by milvus id in Mysql

### 3. Start Client

Next, start the frontend GUI.

- **Set parameters**

Modify the parameters according to your own environment.

| **Parameter**   | **Description**                                       | **example**      |
| --------------- | ----------------------------------------------------- | ---------------- |
| **API_HOST** | The IP address of the backend server.                    | 127.0.0.1        |
| **API_PORT** | The port of the backend server.                          | 5000             |

```bash
$ export API_HOST='127.0.0.1'
$ export API_PORT='5000'
```

- **Run Docker**

First, build a container by pulling docker image.

```bash
$ docker run -d \
-p 8001:80 \
-e "API_URL=http://${API_HOST}:${API_PORT}" \
 milvusbootcamp/img-search-client:1.0
```

## How to use front-end

Navigate to `127.0.0.1:8001` in your browser to access the front-end interface.

### 1. Insert data

Enter `/data` in `path/to/your/images`, then click `+` to load the pictures. The following screenshot shows the loading process:

<img src="pic/web2.png" width = "650" height = "500" alt="arch" align=center />

> Notes:
>
> After clicking the Load (+) button, the first time load will take longer time since it needs time to download and prepare models. Please do not click again.
>
> You can check backend status for progress (check in terminal if using source code OR check docker logs of the server container if using docker)

The loading process may take several minutes. The following screenshot shows the interface with images loaded.

<img src="pic/web3.png" width = "650" height = "500" alt="arch" align=center />

### 2.Search for similar images

Select an image to search.

<img src="pic/web5.png" width = "650" height = "500" alt="arch" align=center />

## Code  structure

If you are interested in our code or would like to contribute code, feel free to learn more about our code structure.

```bash
server
├── Dockerfile
├── requirements.txt
└── src
    ├── __init__.py
    ├── config.py # Configuration file
    ├── encode.py # Convert an image to embedding using towhee pipeline (ResNet50)
    ├── encode_tf_resnet50.py # Old encoder file using ResNet50 by tensorflow
    ├── logs.py
    ├── main.py # Source code to start webserver
    ├── milvus_helpers.py # Connect to Milvus server and insert/drop/query vectors in Milvus.
    ├── mysql_helpers.py # Connect to MySQL server, and add/delete/query IDs and object information.
    ├── operations
    │   ├── __init__.py
    │   ├── count.py
    │   ├── drop.py
    │   ├── load.py
    │   ├── search.py
    │   └── upload.py
    └── test_main.py # Pytest file for main.py
```

![Screenshot from 2022-08-13 01-57-52](https://user-images.githubusercontent.com/66636289/184447727-ec77dc47-25f7-430b-8593-1178683358f0.png)
