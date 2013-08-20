hubot-asgard
============

[Hubot](http://hubot.github.com/) script for interacting with [Asgard](https://github.com/Netflix/asgard)

## Goals
* Allow an easy interface for quick (particularly read-only) data requests from Asgard. 
* Better mobile support via Campfire/Hipchat/XMPP/etc.
* Templated ([eco](https://github.com/sstephenson/eco)) output allows easy customization for users.
* Support basic and/or fundamental updates that should not require a browser to run.
* Additional ACL via Hubot Admin and roles


## Requirements

[Asgard](https://github.com/Netflix/asgard) needs to be running somewhere, and Hubot needs to be able to access it. Hubot-asgard does not allow self-signed SSL certs, so if you are using an un-altered instance based on [these](http://imperialwicket.com/netflix-asgard-12-ami-updates), make sure to hit port 8080 directly, instead of using the SSL proxy. Also note that Hubot-asgard does not currently support basic authentication, another reason to hit 8080 directly.

If you do not have an Asgard instance, you should install both of the asgard.coffee and asgard-launcher.coffee scripts. Asgard-launcher is an AWS-centric launch utility for Asgard. After configuration in Hubot, you can launch a new Asgard instance as follows:

    asgard-launcher run
    asgard-launcher url
    asgard-launcher authorize <HUBOT_IP>
    asgard-launcher authorize <YOUR_IP>

After configuring your Asgard instance (via web browser), you can elect to save a private AMI that includes your configured AWS credentials with:

    asgard-launcher create ami

If you want to shutdown the instance, use:

    asgard-launcher terminate

If you created an ami, asgard-launcher will use that ami for future `asgard-launcher run` requests. If not, it will launch the default ami (requiring configuration) each time.


## Installation

Until hubot-asgard is more mature, it's not going to be available via npm or github/hubot-scripts. None of the github/hubot-scripts use templates and this seems a little problematic; once the directory structure and more of the core functionality is final, I will get things in npm.

Hubot-asgard requires 'eco' (>= 1.1.0) and 'async' (>= 0.2.9). I'm not doing anything cutting edge, and it will probably work with older versions, but I have not tested them. Asgard-launcher is bundled here, and it requires 'aws-sdk' (>=1.5.0) to help with launching and managing the Asgard instance and supporting AWS objects. 

For now, you will need to add the src/scripts/asgard.coffee to your Hubot scripts (and optionally src/scripts/asgard-launcher.coffee) and create the src/templates directory for Hubot. Depending on your configuration, something like this might work:

    HUBOT_DIR=/path/to/hubot/
    cd ~
    git clone https://github.com/imperialwicket/hubot-asgard.git
    cd $HUBOT_DIR
    npm install eco async
    ln -s ~/hubot-asgard/src/scripts/asgard.coffee $HUBOT_DIR/scripts/asgard.coffee
    ln -s ~/hubot-asgard/src/templates/ $HUBOT_DIR/templates


## Configuration options

Hubot-asgard uses Redis to store information via robot.brain. On initial launch, hubot will try to load your HUBOT_ASGARD_URL and HUBOT_ASGARD_REGION from the like-named environment variables. If these are empty, Hubot uses 'http://127.0.0.1' and 'us-east-1'.

You can retrieve and update these values with Hubot via:

    asgard url http://asgard.example.com:8080
    asgard url
    asgard region us-west-2
    asgard region

If you are using asgard-launcher.coffee, you must set AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY environment variables for successful aws-sdk configuration.


## Practical Use

Use `hubot help` or check the asgard.coffee file to get the full list of options with short descriptions. The steps below cover checking autoscaling groups and pushing a new ami to a particular autoscaling group. 

Show autoscaling groups:

    asgard autoscaling
    OR
    a as

Show a single autoscaling group:

    a as autoscaling-group-name

Show amis:

    asgard ami

Start a rolling push:

    asgard rollingpush autoscaling-group-name ami-1234abcd

Check the rolling push task:

    asgard task
    asgard task 12


## Templates

Hubot-asgard returns data via ([eco](https://github.com/sstephenson/eco)) templates. If you are missing data, or want to organize things differently, hit the Asgard web interface that corresponds to a request and append the url with '.json'. This should show you the data that's being passed to the template. Change the template in a fork and either rock your personal changes, or submit a pull request for everyone to enjoy.


## Todo

* Right now the API wrapping is highest priority; mainly next asg creation and asg edits
* Second is nomenclature and making sure that commands make sense and have consistency
* List size checking, response batching. Need basic safety checks in case someone tries to get the list of all AMIs in us-east-1 (or has very large systems).
* Implement roles - Use HUBOT admin, or entirely separate roles? Probably both.
