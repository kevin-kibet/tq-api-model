# Printing Service
Is a service to handle automatic and configurable print jobs. A Frontend is provided to manage and carry out the configurations. Requests are sent in via an HTTP endpoint, and can be monitored, subsequently from the
provided web application.

## Building and Deployment
### Backend
##### Using Jenkins
The Backend is a spring boot application, using gradle as the build system.
Alternatively, you can obtain an already built jar file from Jenkins(Configuring the pipeline)
> Coming soon

##### Building localy
###### Prerequisites
- JDK (At least 1.8)
- Gradle

Clone the project [here](http://10.176.18.27:8080/#/admin/projects/printing-service),
then run this command at the root folder of the project.
Clone the project:
```
git clone [git_url]
```
CD into the directory
```
cd printing-service/backend
```
Build the jar file
```
gradle bootJar
```
Locate the built jar file in the directory **build/libs/**. 

Once you obtain the jar file, transfer it to your server using an FTP client

##### Sample deployment
Deploying to a server with ip **10.176.18.13**.
Copy the jar file to the directory **/u01/applications/Spring** using WinSCP
SSH into the server from the terminal. You can use Putty for windows
Navigate to the directory where the jar file is located
```
cd /u01/applications/Spring
```
Run the project using this command.
```
java -jar -Dspring.profiles.active=test  printing-service-0.0.1-SNAPSHOT.jar &
```
You'll need to specify the active profile using __-Dspring.profiles.active=test__
Available profiles:
 - test
 - staging
 - production

### Frontend
##### Using Jenkins

> coming soon

##### Building locally
###### Prerequisites
- npm
- Angular

The Frontend is an angular application. To build the application, go ahead to clone it [here](http://10.176.18.27:8080/#/admin/projects/printing-service)
Then run these commands at the root folder of the project.
```
npm install
ng build --prod --base-href=[full url of the deployment server + path]
```
To simplify the build process, we're coming up with the configuration to run the tasks using Jenkins.

A folder **dist** is generated with the built files. Copy all the files from the directory,**dist**, to your http server directory

##### Sample deployment
Deploying to  server **10.176.18.13** with an http server(apache or nginx) running on port **8376**.
Apache or nginx installation points to **/var/www/html/** by default
Create a directory to deploy the application
```
mkdir -p /var/www/html/printing-service/
```
Clone the project:
```
git clone [git_url]
```
CD into the directory
```
cd printing-service/frontend
```
Install Dependencies
```
npm install
```
To build, run
```
ng build --prod --base-href=http://10.176.18.13:8376/printing-service/
```
Copy all the files generated in the **dist** directory to the directory you created in the server using an FTP client(e.g WinSCP)


## Integration
The service is available through an http endpoint. You make a call to the end-point to submit a print request: Here's the url.
```
http://[host]:[port]/print-requests
```
#### Request
```
POST /print-requests HTTP/1.1
Host: [host]:[port]
Content-Type: multipart/form-data
Accept: application/json
```

| Field              | Type     | Description                         |
| -------------------|----------| ------------------------------------|
| document           | file     | PDF File to print                   |
| document_reference | string   | Document Reference(e.g Policy Number)|

Sample Code
```java
 public void sendToPrintingService(File file, String reference) {
        try {
            HttpClient httpClient = HttpClientBuilder.create().build();
            // TODO: Dynamic URL from Parameters
            HttpPost httpPost =
                new HttpPost("http://[host]:[port]/print-requests");

            MultipartEntityBuilder builder = MultipartEntityBuilder.create();
            builder.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);

            InputStream inputStream = new FileInputStream(file);

            builder.addBinaryBody("document", inputStream,
                                  ContentType.create("application/pdf"),
                                  file.getName());
            builder.addTextBody("document_reference", reference,
                                ContentType.TEXT_PLAIN);
            //
            HttpEntity entity = builder.build();
            httpPost.setEntity(entity);
            HttpResponse response = httpClient.execute(httpPost);
            int statusCode = response.getStatusLine().getStatusCode();
            if( statusCode == 200 || statusCode == 201) {
              System.out.println("Called Printing Service: Success");
            } else {
              System.out.println("Called Printing Service: Error");
            }
        } catch (FileNotFoundException e1) {
            e1.printStackTrace();
        } catch (Exception e2) {
            e2.printStackTrace();
        }
 }
```
Catch the exception, just in-case the service is down, or the file is missing and report it back to the user.
The end point can also be parameterized using **TQC_PARAMS** 

## TODO
- [x] Setup project in gerrit.
- [ ] Document.
- [ ] Configure Jenkins build pipeline.
