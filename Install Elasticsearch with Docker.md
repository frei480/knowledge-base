Self-Managed

Use Docker commands to start a single-node Elasticsearch cluster for development or testing. You can then run additional Docker commands to add nodes to the test cluster or run Kibana.

> [!Tip]
>
> -   If you just want to test Elasticsearch in local development, refer to [Run Elasticsearch locally](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/local-development-installation-quickstart). Note that this setup is not suitable for production environments.
> -   This setup doesn’t run multiple Elasticsearch nodes or Kibana by default. To create a multi-node cluster with Kibana, use Docker Compose instead. See [Start a multi-node cluster with Docker Compose](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-docker-compose).

# Hardened Docker Images

You can also use the hardened [Wolfi](https://wolfi.dev/) image for additional security. Using Wolfi images requires Docker version 20.10.10 or higher.
To use the Wolfi image, append `-wolfi` to the image tag in the Docker command.

For example:
Latest
```bash
docker pull docker.elastic.co/elasticsearch/elasticsearch-wolfi:9.1.5
```
Specific version

# Start a Single-node Cluster

1.  Install Docker. Visit [Get Docker](https://docs.docker.com/get-docker/) to install Docker for your environment.
    If using Docker Desktop, make sure to allocate at least 4GB of memory. You can adjust memory usage in Docker Desktop by going to **Settings > Resources**.
2.  Create a new docker network.
    ```sh
    docker network create elastic
    ```    
3.  Pull the Elasticsearch Docker image.
    ```bash
    docker pull docker.elastic.co/elasticsearch/elasticsearch:9.1.5
    ```      
    
4.  Optional: Install [Cosign](https://docs.sigstore.dev/cosign/system_config/installation/) for your environment. Then use Cosign to verify the Elasticsearch image’s signature.
    ```bash
    wget https://artifacts.elastic.co/cosign.pub
    cosign verify --key cosign.pub docker.elastic.co/elasticsearch/elasticsearch:9.1.5
    ```    
    The `cosign` command prints the check results and the signature payload in JSON format:

    ```sh
    Verification for docker.elastic.co/elasticsearch/elasticsearch:9.1.5 --
    The following checks were performed on each of these signatures:
      - The cosign claims were validated
      - Existence of the claims in the transparency log was verified offline
      - The signatures were verified against the specified public key
    ```   
    
5.  Start an Elasticsearch container.
    ```sh
    docker run --name es01 --net elastic -p 9200:9200 -it -m 1GB docker.elastic.co/elasticsearch/elasticsearch:9.1.5
    ```

> [!Tip]
> Use the `-m` flag to set a memory limit for the container. This removes the need to [manually set the JVM size](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-docker-prod#docker-set-heap-size).

Machine learning features such as [semantic search with ELSER](https://www.elastic.co/docs/solutions/search/semantic-search/semantic-search-elser-ingest-pipelines) require a larger container with more than 1GB of memory. If you intend to use the machine learning capabilities, then start the container with this command:

```sh
docker run --name es01 --net elastic -p 9200:9200 -it -m 6GB -e "xpack.ml.use_auto_machine_memory_percent=true" docker.elastic.co/elasticsearch/elasticsearch:9.1.5
```

The command prints the `elastic` user password and an enrollment token for Kibana.


6.  Copy the generated `elastic` password and enrollment token. These credentials are only shown when you start Elasticsearch for the first time. You can regenerate the credentials using the following commands.
    ```bash
    docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
    docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
    ```

    We recommend storing the `elastic` password as an environment variable in your shell. Example:

    ```bash
    export ELASTIC_PASSWORD="your_password"
    ```
    
7.  Copy the `http_ca.crt` SSL certificate from the container to your local machine.

    ```sh
    docker cp es01:/usr/share/elasticsearch/config/certs/http_ca.crt .
    ```
    
8.  Make a REST API call to Elasticsearch to ensure the Elasticsearch container is running.

    ```bash
    curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200
    ```
    

# Add More Nodes
1.  Use an existing node to generate a enrollment token for the new node.
    ```bash
    docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
    ```
    The enrollment token is valid for 30 minutes.
2.  Start a new Elasticsearch container. Include the enrollment token as an environment variable.
    ```bash
    docker run -e ENROLLMENT_TOKEN="<token>" --name es02 --net elastic -it -m 1GB docker.elastic.co/elasticsearch/elasticsearch:9.1.5
    ```
    
3.  Call the [cat nodes API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cat-nodes) to verify the node was added to the cluster.
    ```bash
    curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200/_cat/nodes
    ```
    
# Run Kibana
1.  Pull the Kibana Docker image.
    ```bash
    docker pull docker.elastic.co/kibana/kibana:9.1.5
    ```
2.  Optional: Verify the Kibana image’s signature.
    ```bash
    wget https://artifacts.elastic.co/cosign.pub
    cosign verify --key cosign.pub docker.elastic.co/kibana/kibana:9.1.5
    ```
3.  Start a Kibana container.
    ```sh
    docker run --name kib01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:9.1.5
    ```
4.  When Kibana starts, it outputs a unique generated link to the terminal. To access Kibana, open this link in a web browser.
5.  In your browser, enter the enrollment token that was generated when you started Elasticsearch.
    To regenerate the token, run:
    ```bash
    docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
    ```
6.  Log in to Kibana as the `elastic` user with the password that was generated when you started Elasticsearch.
    To regenerate the password, run:
    ```bash
    docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
    ```
    
# Remove Containers
To remove the containers and their network, run:
```bash
# Remove the Elastic network
docker network rm elastic

# Remove Elasticsearch containers
docker rm es01
docker rm es02

# Remove the Kibana container
docker rm kib01
```

# Next Steps
You now have a test Elasticsearch environment set up. Before you start serious development or go into production with Elasticsearch, review the [requirements and recommendations](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-docker-prod) to apply when running Elasticsearch in Docker in production.