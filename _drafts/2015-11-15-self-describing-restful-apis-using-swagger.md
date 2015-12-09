# Self-describing Restful APIs using Swagger 

### Quick background

Until recently, most services at Wonga have been heavily messaging-based NServiceBus endpoints. As we started to roll towards the micro-services approach, we had an increasing number of Restful API endpoints which follow a more Request-Response model. These Restful endpoints are being consumed internally by both Font-End and Back-End teams.

### Problem
One of the frustrations faced by consumers of our APIs often was the absence of documentation; even worse often the documentation would go stale and out of sync as the APIs evolved. Unlike the olden days of SOAP web services, where WSDL served as quite a useful tool for making the service self-descriptive and discoverable; documenting your Rest API contracts hasn't been as seamless as one would hope. 

### Our Solution
In order to address that, a few months ago we started using Swagger to document our Rest APIs in a self-contained way and it has proven to be quite successful across our teams as a way to communicate service contracts for integration. Incidentally it has also been very useful for doing internal demos to Product Owners. 

#### A bit about Swagger
Swagger is an open-source, simple yet powerful and language-agnostic representation of your API, providing interactive documentation and discoverability. With little effort, it can generate the API documentation including HTTP methods, resources, request and response models and HTTP status codes. It also provides a nice sandbox UI you can use as a playground for calling your API. Official page and GitHub repo links below:

[http://swagger.io/](http://swagger.io/ "http://swagger.io/")

[https://github.com/swagger-api/swagger.io](https://github.com/swagger-api/swagger.io)


Here is a screen-shot of how the Swagger-UI looks:

![](/images/2015-11-15-self-describing-restful-apis-using-swagger/swagger-ui-app-man.png)

#### Using Swagger with Nancy Fx 
For a while we were using Nancy Fx to host our Rest API endpoints which had served us quite well. Official page link below:

[http://nancyfx.org/](http://nancyfx.org/ "http://nancyfx.org/")	

Here is some sample code demonstrating a way of having a Docs Module in Nancy :
 
	public class DocsModule : NancyModule
    {
        public DocsModule()
        {
            Get["/"] = _ => RedirectToSwagger();
            Get["/docs"] = _ => RedirectToSwagger();
        }

        private static RedirectResponse RedirectToSwagger()
        {
            return new RedirectResponse("/docs/index.html");
        }
    }

The integration of Nancy with Swagger was a bit fiddly though in a sense that we had to embed the Swagger-ui code as part of our web Host projects. Also the contract definition was specified in a Swagger.json file that needed to be manually kept in-sync with the contract definitions. This posed an issue i-e. whenever we extended the API, we had to remember to update the Json file as well which was prone to human error. Screen-shot below:

![](/images/2015-11-15-self-describing-restful-apis-using-swagger/swagger-json.png)


#### Using Swagger with Web API
As we moved towards using Web API instead of Nancy Fx to host our Rest endpoints, we came across a rather nice open-source implementation from Swashbuckle that allowed auto-generation of Swagger documentation on the fly. It also has an embedded Swagger-UI within the package so you don't have to maintain a copy of the source in your Host projects. Nuget package and GitHub repo links below:

[https://www.nuget.org/packages/Swashbuckle](https://www.nuget.org/packages/Swashbuckle)

[https://github.com/domaindrivendev/Swashbuckle](https://github.com/domaindrivendev/Swashbuckle)
 
Here is some sample code demonstrating a way of having a Docs Controller in Web API:

    public class DocsController : ApiController
    {
        [Route("docs"), HttpGet]
        public HttpResponseMessage Get()
        {
            var redirect = Request.CreateResponse(HttpStatusCode.MovedPermanently);
            redirect.Headers.Location = new Uri(AppSettings.ApiBaseUrl + "docs/index");
            return redirect;
        }
    }

We've been using it successfully for a number of services now and have also managed to extend it somewhat.One particular example is where we used an "OperationFilter" to implement API-key Authorization headers which I'll cover in a separate post. Notice the custom Authorization field on the Swagger-UI in the screen-shot below:

	
![](/images/2015-11-15-self-describing-restful-apis-using-swagger/swagger-ui-emp-details.png)


###Summary

Swashbuckle provides an elegant way to seamlessly integrate Swagger documentation and Swagger-ui to your Rest APIs. It is customizable, extensible and provide out-of box leverage for Xml comments. This keeps the documentation tightly integrated into the code, allowing it to always stay in sync with the contracts.

