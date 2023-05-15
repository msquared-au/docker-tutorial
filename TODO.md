Not in any particular order...
* add notes for setting up an image that has your preferred editor, to simplify editing files in containers
  * add 'vim' to list of requirements, or link to vim docs, or install something other than vim
* add notes about opening/using multiple terminals, since sometimes you'll need multiple containers running
* consider creating a terminology section or terminology document
* consider adding explanatory notes to docker command arguments as they are introduced
  * maybe inline, maybe links to terminology section/document or to docker docs
* add explanatory notes about commandline arguments under "test the nginx image"
* compare ubuntu-supplied docker to docker-supplied docker
  * (ie: do we need to install Docker via https://docs.docker.com/engine/install/ubuntu/ ?)
* check if SSL Mozilla modern profile is valuable (SSL_POLICY=Mozilla-Modern in https://hub.docker.com/r/nginxproxy/nginx-proxy)
  * oddly enough, using the modern profile seems to reduce the site's rating at https://www.ssllabs.com/ssltest/
* consider adding external logging (logging to a volume or to external service)
* advanced: investigate whether multiple docker-compose projects can talk to
  one another and still be supported transparently by an nginx proxy with its
  acme companion:
  * set up one project for nginx and acme
  * set up each application in its own project
  * set up a common network that allows each application project to be
    reachable from the proxy project
  * this is probably the ideal end-goal, since at least some database-backed
    applications seem to be set up as their own docker project with the web
    front-end in one container and the database in another
