# Setup Database

GitLab supports the following databases:

* MySQL (preferred)
* PostgreSQL


## MySQL

    # Install MySQL
    portmaster databases/mysql55-server
    
    # Enable MySQL rc.d start
    
    echo "mysql_enable=YES" >> /etc/rc.conf

    # Login to MySQL, password is blank
    mysql -u root -p

	# After MySQL has start you need to set passwords on
	# both root accounts and the two anonymous accounts. 
	# By default these are left blank and give full access 
	# to the database server to anyone.
	
	# Set a password on the anonymous accounts use. (change $password to a real password)
	
	mysql -u root
	mysql> SET PASSWORD FOR ''@'localhost' = PASSWORD('$password');
	
	# Set a password for the root account. (change $password to a real password)
	
	mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('$password');

    # Create a user for GitLab. (change $password to a real password)
    
    mysql> CREATE USER 'gitlab'@'localhost' IDENTIFIED BY '$password';

    # Create the GitLab production database
    mysql> CREATE DATABASE IF NOT EXISTS `gitlabhq_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;

    # Grant the GitLab user necessary permissopns on the table.
    mysql> GRANT SELECT, LOCK TABLES, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON `gitlabhq_production`.* TO 'gitlab'@'localhost';

    # Quit the database session
    mysql> \q

    # Try connecting to the new database with the new user
    su -m git -c "mysql -u gitlab -p -D gitlabhq_production"

## PostgreSQL

    # Install the PostgreSQL
    portmaster databases/postgresql92-server
	
	# Enable PostgresSQL rc.d start
	
	echo "postgresql_enable="YES"" >> /etc/rc.conf
	
	# Initialize the database
	
	/usr/local/etc/rc.d/postgresql initdb
	
	# Start PostgreSQL
	
	service postgresql start
	
    # Login to PostgreSQL
    su -m pgsql -c "psql template1"

    # Create a user for GitLab. (change $password to a real password)
    template1=# CREATE USER git WITH PASSWORD '$password';

    # Create the GitLab production database & grant all privileges on database
    template1=# CREATE DATABASE gitlabhq_production OWNER git;

    # Quit the database session
    template1=# \q

    # Try connecting to the new database with the new user
    su -m git -c "psql -d gitlabhq_production"
	
	# Quit the database test session
    template1=# \q
