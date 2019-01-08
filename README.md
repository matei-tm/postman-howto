# postman-howto
Postman notes

## Get custom content from a REST service

### The problem
We have to solve the following problem:

A REST service with a base URI http://seismic-alert.ro/posts/ returns "posts" JSON elements. The first n elements must be transformed to a lighter custom JSON format. The collection with the custom JSON format must be returned.

#### Original format

The original elements are in the following format:

```JSON
{
"id":1,
"name":null,
"address":"Strada ACADEMIEI",
"lat":"44.436402",
"lng":"26.099322",
"author_id":1,
"approver_id":null,
"approved":null,
"approved_at":null,
"text":null,
"created_at":"2016-05-22T21:01:33.000Z",
"updated_at":"2016-09-01T19:02:03.000Z",
"sector":1,
"city":"București",
"upload_id":1,
"active_submission_id":2602,
"uploaded_submission_id":1,
"street_number":"15",
"submission_count":2,
"active_submission":
	{
		"id":2602,
		"appartments_no":6,
		"occupied_appartments_no":3,
		"residents_no":7,
		"post_id":1,
		"created_at":"2016-09-01T19:00:41.000Z",
		"updated_at":"2016-09-01T19:00:41.000Z",
		"has_owners_association":false,
		"debtors_no":null,
		"utilities_disconnections_comments":"",
		"has_tehnical_issues_comments":"",
		"owners_no":6,
		"public_owners_no":null,
		"public_owners_comments":"",
		"rented_appartments_no":null,
		"offices_no":null,
		"commercial_spaces_no":null,
		"neighbouring_residential":null,
		"neighbouring_commercial_or_cultural":null,
		"neighbouring_public_buildings":null,
		"neighbouring_public_space":null,
		"neighbouring_yard":null,
		"neighbouring_comments":"In cladire se afla teatrul Odeon",
		"source_name":"",
		"source_phone_number":"",
		"source_email":"",
		"seismic_class_id":1,
		"import_originated":null,
		"accepted":null,
		"height_standard":null,
		"construction_year":null,
		"surface":null,
		"technical_evaluation_year":null,
		"technical_evaluator_name":null,
		"building_still_exists":true,
		"owned_by_companies_no":null,
		"personal_opinion":"",
		"rehabilitation_variant_id":3,
		"submitter_name":"Anghene Laura si Bogdan Suditu",
		"submitter_phone_number":"0769289483",
		"submitter_email":"laurageorgiana.anghene@gmail.com",
		"images":[],
		"created_by_user_id":3,
		"seismic_class":
			{
				"id":1,
				"acronym":"rs i - pericol public",
				"name":"Rs I PP",
				"description":"Lista imobilelor expertizate tehnic din punct de vedere al riscului seismic încadrate în clasa I de risc seismic care prezinta pericol public  ( clasa RsI. din care fac parte construcțiile cu risc ridicat de prăbușire la cutremurul de proiectare corespunzător stării limita ) (conform Codului de evaluare antiseismica P100-3/2008)",
				"created_at":"2016-05-22T20:30:48.000Z",
				"updated_at":"2016-06-12T11:51:28.000Z",
				"long_name":"Clasa I - Pericol Public de risc seismic",
				"cnt":0,
				"image_marker":"markers/darkest_red.png"
			}
	}
}					
```

#### Output format

```JSON
{
    "id":1,
    "address":"Strada ACADEMIEI",
    "lat":"44.436402",
    "lng":"26.099322",
    "street_number":"15",
    "appartments_no":6,
    "occupied_appartments_no":3,
    "seismic_class_id":1,
    "source_name":"",
    "source_phone_number":""}
```

### Solutions

### Solution 1. GET the entire collection in a single step

Get the entire collection of the elements from http://seismic-alert.ro/posts/ and transform it to the new collection of custom objects

### Solution 2. GET entity by entity and transform

An exotic solution that could be applied to any service that do not expose an endpoint to reach entire collection and must be visited entity by entity.

We will access and iterate the REST service element by element and extract custom content as new JSON.

#### Add new environment 

![new environment](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/00_new_environment.png)

#### Create the variables

![create variables](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/01_new_environment.png)

#### Add new collection

![new collection](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/00_new_collection.png)

#### Create the collection

![save the collection](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/01_new_collection.png)

#### Add request

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

#### Run with collection runner
![select environment](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/07_select_environment.png)
![run](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/08_run.png)
![show console](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/09_show_console.png)

![execute](https://github.com/matei-tm/postman-howto/raw/master/rest_iteration/images/10_execute.png)
