# Use the official image as a parent image
FROM tomcat:8.0

# Set the working directory in the container
WORKDIR /usr/local/tomcat

# Copy the WAR file into the webapps directory of Tomcat
COPY /path/to/your/app.war /usr/local/tomcat/webapps/

# Make port 8080 available to the world outside this container
EXPOSE 8080

# Run the command # Finally, we run the catalina.sh script with the run command to start Tomcat.
CMD ["catalina.sh", "run"]  