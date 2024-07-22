### Crypto Detector with DIY AWS Lambda for PHP Applications

**Project Overview:**

This project aims to develop a robust crypto detector that identifies and mitigates suspicious cryptocurrency activities and integrates a DIY AWS Lambda solution for PHP applications. The combined system leverages advanced cybersecurity practices to enhance threat detection and application security in the cloud.

**Key Features:**

- **Crypto Detector:**
  - **Cryptocurrency Threat Detection:** Implements machine learning algorithms and blockchain analysis tools to identify fraudulent activities, money laundering, and other cyber threats in the cryptocurrency ecosystem.
  - **Blockchain Security Analysis:** Evaluates the security of blockchain networks and smart contracts, detecting vulnerabilities and providing protective measures to ensure secure transactions.

- **DIY AWS Lambda for PHP Applications:**
  - **Serverless Architecture:** Utilizes AWS Lambda to run PHP applications in a serverless environment, reducing infrastructure management and enhancing scalability.
  - **Automated Security Updates:** Ensures PHP applications receive timely security updates, reducing exposure to vulnerabilities.
  - **Secure API Integration:** Facilitates secure communication between Lambda functions and other AWS services, adhering to best practices for encryption and authentication.

**Advanced Cybersecurity Skills:**

1. **Cryptocurrency Threat Detection:**
   - Expertise in analyzing cryptocurrency transactions for suspicious activities, using blockchain analysis tools to identify and prevent fraud.

2. **Blockchain Security Analysis:**
   - Proficient in securing blockchain networks and smart contracts, conducting thorough security assessments to safeguard against cyber threats.

**How It Works:**

1. **Crypto Detector:**
   - **Data Collection:** Collects and analyzes cryptocurrency transaction data from various sources.
   - **Threat Detection:** Uses machine learning to identify patterns and anomalies indicating potential threats.
   - **Reporting:** Provides detailed reports on detected threats and recommended actions for mitigation.

2. **AWS Lambda Integration:**
   - **Lambda Deployment:** Deploys PHP applications as Lambda functions, enabling serverless execution and scalability.
   - **API Gateway:** Integrates with AWS API Gateway to handle requests securely, utilizing IAM roles and policies for access control.
   - **Continuous Monitoring:** Monitors Lambda functions for performance and security, ensuring compliance with cybersecurity standards.

**Getting Started:**

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/your-username/crypto-detector-aws-lambda-php.git
   ```
2. **Setup AWS Lambda:**
   - Follow the instructions in the `docs/aws-lambda-setup.md` to configure and deploy your PHP applications using AWS Lambda.

3. **Configure Crypto Detector:**
   - Refer to `docs/crypto-detector-setup.md` for steps to set up and configure the crypto detector module.

4. **Run Security Assessments:**
   - Use provided scripts and tools in the `security-tools` directory to perform blockchain security analysis and cryptocurrency threat detection.

**Documentation:**
- **Setup Guides:** Detailed instructions for setting up the crypto detector and AWS Lambda environment.
- **API Reference:** Documentation for API endpoints and integration details.
- **Security Best Practices:** Guidelines on maintaining a secure deployment and responding to detected threats.

**Contributing:**
- **Issues and Features:** Submit issues and feature requests through the GitHub Issues page.
- **Pull Requests:** Contribute by creating pull requests with enhancements or bug fixes.

**License:**
- This project is licensed under the [MIT License](LICENSE).

---

This description provides a comprehensive yet concise overview of the project, focusing on both the crypto detector and the DIY AWS Lambda integration for PHP applications.


DIY AWS Lambda for PHP applications!

Docker image containing Ubuntu 22.04 LTS core with Apache 2.4 and PHP 8.1. This image is designed to be used in AWS environments for high density PHP application hosting. WordPress 5.x and Drupal 7.x are tested to work.

# Architecture Overview

* Run multiple EC2 instances across different availability zones to create a redundant docker swarm.
* Cloudwatch alarms must be used to restart failed EC2 instances.
* All EC2 instances must join the docker swarm and mount a common EFS volume on `/srv`.
* Docker container will mount `/srv/example.com/www` as the Apache `DocumentRoot` to serve php applications.
* RDS must be used for hosting databases.
* When this image is run as a docker service in the swarm, a unique port on the host (eg tcp:8001) is mapped to 80 within the container.
* AWS Target Group `example-com` is created with all the EC2 instances of the swarm and specific TCP port (eg tcp:8001) and attached to ALB.
* Docker mesh routing is not cookie/sticky session aware and disabled. HTTP load balancing is fully managed on AWS ALB.
* AWS ALB rules are used to route example.com and www.example.com requests to `example-com` Target Group.
* AWS ALB is also used for HTTPS termination, docker containers provide only vanilla HTTP.
* Outbound emails must be routed via SES (Drupal SMTP module, WP SMTP plugin, etc).
* Apache logs are sent to stdout/stderr and can be routed to AWS Cloudwatch using the docker `awslogs` log driver.

# Filesystem Layout

* `/srv` -- base dir to be published via AWS EFS to all nodes, subdirectories `/srv/example.com`, `/srv/example.net`, etc to be created for each website/domain.
* `/srv/example.com/www` -- root folder for php applications (WordPress root, Drupal root, etc), mounted within docker containers as Apache `DocumentRoot`
* `/srv/example.com/etc/apache` and `/srv/example.com/etc/php` -- optional apache/php config
* `/srv/example.com/mysqlbackup` -- used for storing MySQL dumps

# Small But Significant Things

* Apache MPM prefork is configured to reduce RAM usage, a WordPress 5.x container will idle around 50-75MB of RAM.
* WordPress W3TC plugin cache folder is in `$WP_ROOT/wp-content/cache`. EFS is slow, so it's recommended to move the cache folder inside the docker container. Delete the cache folder and run `ln -s /srv/example.com/www/wp-content/cache /tmp`. Cache folder will be hosted within docker instance and run faster.
* PHP POST max and max file upload size is increased to 64MB to handle large file uploads.
* EFS is slow, to improve performance php opcache revalidation time is increased from default 2 seconds to 300 seconds.
* apache process runs as `www-data`. Inside the docker instance, none of the processes are run as `root`. `setcap` is used to permit non-root apache process to bind to TCP port 80.
* Apache config is aware of SSL termination on AWS ALB or CloudFlare Flexible SSL settings. WordPress site URL can be set to https://www.example.com and SSL termination can be done on AWS ALB or CloudFlare and WordPress will not throw infinite HTTP redirection errors.
* To view the default Apache and PHP config, run the docker image without mapping an external DocumentRoot and access the container via http://localhost:8001/index.php (returns phpinfo) and http://localhost:8001/.config (contains /etc/apache and /etc/php tar files). This can be used for further customization.
* To use custom Apache config, extract the base apache config in `/svr/example.com/etc` and then bind mount it inside the container. Eg. `docker run -v /srv/example.com/etc/php:/etc/php:ro -v /srv/example.com/etc/apache:/etc/apache:ro ...`
* WordPress updates can be handled outside the docker environment via WP-CLI. From an independent EC2 instance, 1) apt install all Apache/PHP WordPress dependencies, 2) mount EFS /srv volume, 3) run `cd /srv/example.com/www && wp-cli plugin update --all`.

# Building

To build:

```bash
docker build --no-cache -t rsubr/php-apache-ubuntu:jammy .
```

# Running

## Example 1: Basic usage

Run with internal document root to reveal Apache/PHP config. See http://localhost/index.php and http://localhost/.config/

```bash
docker run --name=test -p 80:80 rsubr/php-apache-ubuntu:jammy
```

## Example 2: Testing WordPress

Run a WordPress site from /srv/example.com/www

```bash
docker run --name=example-com -v /srv/example.com/www:/var/www/html -p 80:80 rsubr/php-apache-ubuntu:jammy
```

## Example 3: Using custom apache2 and php config

Run a WordPress site from /srv/example.com/www, but this time use custom apache and php config from /srv/example.com/etc/{apache,php}

```bash
docker run --name=example-com -v /srv/example.com/www:/var/www/html -v /srv/example.com/etc/apache2:/etc/apache2:ro -v /srv/example.com/etc/php:/etc/php:ro -p 80:80 rsubr/php-apache-ubuntu:jammy
```

## Example 4: Running as a docker service in a docker swarm

Run 2 replicas of the container as a docker service. This command must be run from a docker swarm manager node. AWS ALB and Target Group must be created to route traffic for example.com to this container:

```bash
docker service create --replicas 2 --name example-com --publish published=8000,target=80,mode=host --mount type=bind,source=/srv/example.com/www,destination=/var/www/html rsubr/php-apache-ubuntu:jammy
```

# TODO

* Instead of AWS ECS, this setup uses portainer.io or other docker swarm manager.
* Autoscaling is not possible with current architecture, needs ECS.
* Improve documentation.
* Documentation for EFS issues and workaround using syncthing. 
