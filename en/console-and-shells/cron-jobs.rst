Running Shells as Cron Jobs
###########################

A common thing to do with a shell is making it run as a cronjob to
clean up the database once in a while or send newsletters. This is
trivial to setup, for example::

      */5  *    *    *    *  cd /full/path/to/root && bin/cake myshell myparam
    # *    *    *    *    *  command to execute
    # │    │    │    │    │
    # │    │    │    │    │
    # │    │    │    │    \───── day of week (0 - 6) (0 to 6 are Sunday to Saturday,
      |    |    |    |           or use names)
    # │    │    │    \────────── month (1 - 12)
    # │    │    \─────────────── day of month (1 - 31)
    # │    \──────────────────── hour (0 - 23)
    # \───────────────────────── min (0 - 59)

You can see more info here: http://en.wikipedia.org/wiki/Cron

.. meta::
    :title lang=en: Running Shells as cronjobs
    :keywords lang=en: cronjob,bash script,crontab

Example
#######
In this example will be creating Cron task for automatically sending email to users saved in database. We will be using Linux as our operating system. If you are a window user you can create schedule as well there is help available on how to create automatic task using window in the following link.
http://www.web-site-scripts.com/knowledge-base/article/AA-00487/0/Setup-Cron-job-on-Windows-7-Vista-2008.html

Create Table
###########

CREATE TABLE members (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR (100),
    created DATETIME DEFAULT NULL,
    modified DATETIME DEFAULT NULL
);

Create Model
############
Create a Model name MembersTable inside /src/Model/Table/ folder as follows
```
// src/Model/Table/MembersTable.php

namespace App\Model\Table;

use Cake\ORM\Table;

class MembersTable extends Table
{
    public function initialize(array $config)
    {
        $this->addBehavior('Timestamp');
    }
}
```
Create a Shell Script
#####################
Console/Shell applications are ideal for handling a variety of background tasks. Before we get into specifics, let’s make sure you can run the CakePHP console. First, you’ll need to bring up a system shell. Open command prompt or bash shell and goto to 

$ cd /path/to/app
$ bin/cake

For Windows, the command needs to be bin\cake (note the backslash).
Running the Console with no arguments produces this help message:
Welcome to CakePHP v3.0.0 Console
---------------------------------------------------------------
App : App
Path: /Users/markstory/Sites/cakephp-app/src/
---------------------------------------------------------------
Current Paths:

 -app: src
 -root: /Users/markstory/Sites/cakephp-app
 -core: /Users/markstory/Sites/cakephp-app/vendor/cakephp/cakephp

Changing Paths:

Your working path should be the same as your application path. To change your path use the '-app' param.
Example: -app relative/path/to/myapp or -app /absolute/path/to/myapp

Available Shells:

[CORE] bake, i18n, server, test

[app] behavior_time, console, orm

To run an app or core command, type cake shell_name [args]
To run a plugin command, type cake Plugin.shell_name [args]
To get help on a specific command, type cake shell_name --help

If you see this message we are good to go.
Let’s create a shell for use in the Console. For this example, we’ll create email sending script shell. In your application’s Shell directory create SendEmailShell.php. Put the following code inside it:
namespace App\Shell;

use Cake\Console\Shell;

class MailerShell extends Shell
{
    public function sendEmail()
    {
        $this->out('Does nothing');
    }
}

Save it and From your application directory, run:
bin/cake mailer send_email

You should see the output “Does Nothing”. 
Things to note when running the script the class name MailerShell is converted into lower case and the Shell postfix is removed, similarly the method name is also lower case and the two words are separated by ‘_’ underscore.

Load Models in Your Shells
##########################
To get all the email from database we will use the Model class we created earlier. Load the Model in a Shell as follows
namespace App\Shell;

use Cake\Console\Shell;

class MailerShell extends Shell
{

    public function initialize()
    {
        parent::initialize();
        $this->loadModel('Users');
    }

    public function sendEmail()
    {
        $this->out(‘Does Nothing’);
    }
}

The initialize() method will load the Users Model with which we can fetch all the emails and from database and send email to.
Sending Email
To send email we will use cakephp 3.0 new Email class. Modify our script as follows
namespace App\Shell;

use Cake\Console\Shell;
use Cake\Network\Email\Email;

class MailerShell extends Shell
{

    public function initialize()
    {
        parent::initialize();
        $this->loadModel('Users');
    }

    public function sendEmail()
    {
	$emails = $this->Users->find('all', [
           	 	'fields' => ['email']
       	 ]);

        
     
$email = new Email('default');
      foreach($emails as $email){
	$email->from(['myemail@example.com' => 'My Site'])
    	->to($email)
    	->subject('Your subject')
    	->send('Message body');

            	}
    }
}

Running the script
##################
To run the script automatically in background, put the shell script we just created in one of these folders: /etc/cron.daily, /etc/cron.hourly, /etc/cron.monthly or /etc/cron.weekly. I would like to put this script in cron.weekly folder as I would like to inform my users about weekly changes I do.
That’s it, it’s that how simple is to create Shell script in cakephp 3.0.
