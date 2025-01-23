# WordPress Vagrant Setup

This project sets up a WordPress environment using Vagrant.

## Requirements

- Vagrant
- VirtualBox

## Setup

1. Clone the repository:
       ```sh
       git clone https://your-repo-url.git
       cd your-repo-directory

    #Copy the example config file and update it with your settings:

       cp config.yml.example config.yml

    #Note: You must rename config.yml.example to config.yml and set your MySQL password in the file.

    #If you have a database dump file, rename wordpress_dump.sql.example to wordpress_dump.sql and place it in the project directory. This step is optional and can be ignored if you do not have a dump file.

    #Start the Vagrant environment:

       vagrant up

2. Configuration

    config.yml: Contains the MySQL password for the WordPress database.
    wordpress_dump.sql: (Optional) Your database dump file for initializing the WordPress database.

3. Usage

    Access WordPress at http://192.168.66.150

4. Troubleshooting

    If you encounter any issues, check the Vagrant logs and ensure all dependencies are installed.
