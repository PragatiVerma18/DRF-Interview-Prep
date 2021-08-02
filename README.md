# DRF Interview Questions & Answers

> Click :star: if you like the project. Pull Request are highly appreciated. Follow me [@pragati_verma18](https://twitter.com/pragati_verma18) for technical updates.

---
<div align="center">
<a href="https://twitter.com/pragati_verma18" target="_blank">
<img src=https://img.shields.io/badge/twitter-%2300acee.svg?&style=for-the-badge&logo=twitter&logoColor=white alt=twitter style="margin-bottom: 5px;" />
</a>
<a href="https://dev.to/pragativerma18" target="_blank">
<img src=https://img.shields.io/badge/dev.to-%2308090A.svg?&style=for-the-badge&logo=dev.to&logoColor=white alt=devto style="margin-bottom: 5px;" />
</a>
  <a href="https://pragativerma18.hashnode.dev/" target="_blank">
    <img src="https://img.shields.io/badge/Hashnode-2962FF?style=for-the-badge&logo=hashnode&logoColor=white"/>
  </a>
<a href="https://linkedin.com/in/pragativerma18" target="_blank">
<img src=https://img.shields.io/badge/linkedin-%231E77B5.svg?&style=for-the-badge&logo=linkedin&logoColor=white alt=linkedin style="margin-bottom: 5px;" />
</a>
</div>

---

## Core Concepts


    
1. ### What is an API?
   An API is a set of definitions and protocols for building and integrating application software. API stands for **Application Programming Interface**. APIs let your product or service communicate with other products and services without having to know how they’re implemented. This can simplify app development, saving time and money. The API is not the database or even the server; it’s the code that governs the access point(s) for the server. 

   It’s sometimes referred to as a contract between an information provider and an information user—establishing the content required from the consumer (the call) and the content required by the producer (the response). For example, the API design for a weather service could specify that the user supply a zip code and that the producer reply with a 2-part answer, the first being the high temperature, and the second being the low. 


   **[⬆ Back to Top](#table-of-contents)**

2. ### What is a web API?
   A web API is a collection of endpoints that expose certain parts of an underlying database. As developers we control the URLs for each endpoint, what underlying data is available, and what actions are possible via HTTP verbs.


   **[⬆ Back to Top](#table-of-contents)**

3. ### What is a REST API?
   A **REST(Representational State Transfer) API** (also known as RESTful API) is an API that conforms to the constraints of REST architectural style and allows for interaction with RESTful web services. When a client request is made via a RESTful API, it transfers a representation of the state of the resource to the requester or endpoint. 

   Every RESTful API:

   - is stateless, like HTTP
   - supports common HTTP verbs (GET, POST, PUT, DELETE, etc.)
   - returns data in either the JSON or XML format

   Any RESTful API must, at a minimum, have these three principles. The standard is important because it provides a consistent way to both design and consume web APIs.

   **[⬆ Back to Top](#table-of-contents)**

4. ### What is an endpoint?
   A web API has endpoints - URLs with a list of available actions (HTTP verbs) that expose data (typically in JSON, which is the most common data format these days and the default for Django REST Framework).

   The type of endpoint which returns multiple data resources is known as a **collection.**

   **[⬆ Back to Top](#table-of-contents)**

5. ### What are HTTP Verbs?
   HTTP defines a set of request methods to indicate the desired action to be performed for a given resource. Although they can also be nouns, these request methods    are sometimes referred to as HTTP verbs.
   ![HTTP Verbs](https://user-images.githubusercontent.com/42115530/104155356-08919a00-540d-11eb-94f8-316e8f591177.png)

   **[⬆ Back to Top](#table-of-contents)**

6. ### What is the difference between HTTP and HTTPS?
   HTTPS stands for Hypertext Transfer Protocol Secure (also referred to as HTTP over TLS or HTTP over SSL). HTTPS also uses TCP (Transmission Control Protocol) to send and receive data packets, but it does so over port 443, within a connection encrypted by Transport Layer Security (TLS). Generally sites running over HTTPS will have a redirect in place so even if you type in `http://` it will redirect to deliver over a secured connection.

   **Key Differences:**

  - HTTP is unsecured while HTTPS is secured.
  - HTTP sends data over port 80 while HTTPS uses port 443.
  - HTTP operates at application layer, while HTTPS operates at transport layer.
  - No SSL certificates are required for HTTP, with HTTPS it is required that you have an SSL certificate and it is signed by a CA.
  - HTTP doesn't require domain validation, where as HTTPS requires at least domain validation and certain certificates even require legal document validation.
  - No encryption in HTTP, with HTTPS the data is encrypted before sending.

   **[⬆ Back to Top](#table-of-contents)**

7. ### What are status codes?
   HTTP response status codes indicate whether a specific HTTP request has been successfully completed. Responses are grouped in five classes:
  - **1xx: Informational** – Communicates transfer protocol-level information.
  - **2xx: Success** – Indicates that the client’s request was accepted successfully.
  - **3xx: Redirection** – Indicates that the client must take some additional action in order to complete their request.
  - **4xx: Client Error** – This category of error status codes points the finger at clients.
  - **5xx: Server Error** – The server takes responsibility for these error status codes.

   **[⬆ Back to Top](#table-of-contents)**
   
8. ### What is the difference between authentication and authorization?
   Authentication confirms that users are who they say they are. Authorization gives those users permission to access a resource. In secure environments, authorization must always follow authentication. Users should first prove that their identities are genuine before an organization’s administrators grant them access to the requested resources.
   
   Let's use an analogy to outline the differences.

   Consider a person walking up to a locked door to provide care to a pet while the family is away on vacation. That person needs:

   Authentication, in the form of a key. The lock on the door only grants access to someone with the correct key in much the same way that a system only grants access to users who have the correct credentials.
   Authorization, in the form of permissions. Once inside, the person has the authorization to access the kitchen and open the cupboard that holds the pet food. The person may not have permission to go into the bedroom for a quick nap. 
   
   **[⬆ Back to Top](#table-of-contents)**
   
9. ### What is a browsable API?
   Django REST Framework supports generating human-friendly HTML output for each resource when the HTML format is requested. These pages allow for easy browsing of resources, as well as forms for submitting data to the resources using POST, PUT, and DELETE. It facilitates interaction with RESTful web service through any web browser. To enable this feature, we should specify text/html for the Content-Type key in the request header.

   **[⬆ Back to Top](#table-of-contents)**

10. ### What is CORS?
    Cross-Origin Resource Sharing (CORS) is a protocol that enables scripts running on a browser client to interact with resources from a different origin. This is useful because, thanks to the same-origin policy followed by XMLHttpRequest and fetch, JavaScript can only make calls to URLs that live on the same origin as the location where the script is running.

   **[⬆ Back to Top](#table-of-contents)**
   
11. ### How to fix CORS error in Django?
    CORS requires the server to include specific HTTP headers that allow for the client to determine if and when cross-domain requests should be allowed.

    The easiest way to handle this–-and the one recommended by Django REST Framework–-is to use middleware that will automatically include the appropriate HTTP headers based on our settings.

    We use `django-cors-headers`:

    - add `corsheaders` to the `INSTALLED_APPS`
    - add `CorsMiddleware` above `CommonMiddleWare` in `MIDDLEWARE`
    - create a `CORS_ORIGIN_WHITELIST` 
   
   
   **[⬆ Back to Top](#table-of-contents)**
   
---

## Django Rest Framework

1. ### What is Django Rest Framework?
   Django REST Framework is a web framework built over Django that helps to create web APIs which are a collection of URL endpoints containing available HTTP verbs that return JSON. It’s very easy to build model-backed APIs that have authentication policies and are browsable.
   
   **[⬆ Back to Top](#table-of-contents)**

2. ### What are benefits of using Django Rest Framework?
   - Its Web-browsable API is a huge usability win for your developers.
   - Authentication policies include packages for OAuth1 and OAuth2.
   - Serialization supports both ORM and non-ORM data sources.
   - It’s customizable all the way down. Just use regular function-based views if you don’t need the more powerful features.
   - It has extensive documentation and great community support.
   - It’s used and trusted by internationally recognized companies including Mozilla, Red Hat, Heroku, and Eventbrite.
 
   **[⬆ Back to Top](#table-of-contents)**
 
 3. ### What are serializers?
    Serializers allow complex data such as querysets and model instances to be converted to native Python datatypes that can then be easily rendered into JSON, XML or other content types. Serializers also provide deserialization, allowing parsed data to be converted back into complex types, after first validating the incoming data.

   **[⬆ Back to Top](#table-of-contents)**
   
4. 
 

    
    
    

