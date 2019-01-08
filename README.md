# postman-howto
Postman notes

## Get custom content from a REST service

Quick tutorial to access and iterate a REST service that return JSON response and extract custom content as new JSON.

### Add new environment 

![new environment](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/00_new_environment.png)

### Create the variables

![create variables](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/01_new_environment.png)

### Add new collection

![new collection](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/00_new_collection.png)

### Create the collection

![save the collection](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/01_new_collection.png)

### Add request

![add request](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/02_add_request.png)
![save request](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/03_save_request.png)

![enter url variable](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/04_request_url.png)

```javascript
var loop_id = Number(postman.getEnvironmentVariable("counterId"));
var loop_maxval = Number(postman.getEnvironmentVariable("counterMaxval"));

if (!loop_id || loop_id > loop_maxval) 
{
    loop_id = 1;
}

pm.variables.set("url", "http://seismic-alert.ro/posts/" + loop_id)
postman.setEnvironmentVariable("counterId" , loop_id + 1);
```

![add the script](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/05_prerequestscript.png)

```javascript
var jsonData = JSON.parse(responseBody);

var old = pm.environment.get("responseData");
old = JSON.parse(old);

var filteredJsonData = {};

filteredJsonData.id = jsonData.id;
filteredJsonData.address = jsonData.address;
filteredJsonData.lat = jsonData.lat;
filteredJsonData.lng = jsonData.lng;
filteredJsonData.street_number = jsonData.street_number;
filteredJsonData.appartments_no = jsonData.active_submission.appartments_no;
filteredJsonData.occupied_appartments_no = jsonData.active_submission.occupied_appartments_no;
filteredJsonData.seismic_class_id = jsonData.active_submission.seismic_class.id;
filteredJsonData.source_name = jsonData.active_submission.source_name;
filteredJsonData.source_phone_number = jsonData.active_submission.source_phone_number;

old.push(filteredJsonData);
old = JSON.stringify(old);
pm.environment.set("responseData", old);

var current_id = Number(postman.getEnvironmentVariable("counterId"));

if (current_id > Number(postman.getEnvironmentVariable("counterMaxval"))) 
{
    console.log(old);
}
```

![tests content](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/06_tests.png)

### Run with collection runner
![select environment](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/07_select_environment.png)
![run](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/08_run.png)
![show console](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/09_show_console.png)

![execute](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/10_execute.png)
