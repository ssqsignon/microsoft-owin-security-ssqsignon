# microsoft-owin-security-ssqsignon

[OWIN](http://owin.org/) middleware that enables an application to support the [SSQ signon](https://ssqsignon.com) authentication workflow.

This module lets you authenticate HTTP requests using the 
*SSQ signon online authorization server* access tokens in your ASP.Net web API
applications. Access tokens are typically used to protect API endpoints.

By plugging into OWIN, *SSQ singon* authentication support can be easily and unobtrusively
integrated into any application or framework.

## Install

    $ nuget install Microsoft.Owin.Security.SSQSignon

## Usage

#### Configure Strategy

The SSQ signon authentication middleware authenticates users using an access
token generated by the *token endpoint* of your SSQ signon module.
Once authenticated, the *user id* and *scope* contained within the token are transformed into a `ClaimsIdentity` object.
The strategy requires your module's `module name`.

    // Startup.Auth.cs
    public void ConfigureAuth(IAppBuilder app)
    {
        app.UseSSQSignonAuthentication("Your-SSQSignon-module-name");
    }

#### Authenticate Requests

The *user id* inside contained inside the token will be stored in the User.Identity.Name property of the Controller.
Each space separated component of the *scope* inside the token will be translated into a *Role* of the identity.
You may then use the roles to seamlessly apply permissions into your application.

For example:

    [SSQSignonAuthentication, Authorize(Roles="cat")]
    public class CatController : ApiController
    {
        public dynamic Get()
        {
            return Json(new { message = string.Format("Hello {0}, you are a cat!", this.User.Identity.Name) });
        }
    }
    
#### Global filter

If you want all of your controllers to authenticate using SSQ signon, instead of adding the *SSQSignonAuthentication* attribute to each one you may simply add the provided filter globally.

    // WebApiConfig.cs
    public static void Register(HttpConfiguration config)
    {
        config.SuppressDefaultHostAuthentication();
        config.Filters.Add(new SSQSignonAthenticationFilter());

        // Web API routes
        config.MapHttpAttributeRoutes();

        config.Routes.MapHttpRoute(
            name: "DefaultApi",
            routeTemplate: "{controller}/{id}",
            defaults: new { id = RouteParameter.Optional }
        );
    }

#### Issuing Tokens

For details on how to issue access tokens with the *SSQ signon token endpoint* please visit [ssqsignon.com](https://ssqsignon.com)

## How it works

The middleware dispatches the received token as an HTTPS request to the *token validation endpoint* of your *SSQ signon* module.
If the request is successful, the resulting JSON is parsed, and a `ClaimsIdentity` object is created.
The *user id* is set as the `NameIdentifier` claim, while each whitespace separated segment in the *scope* is added as a `Role` claim.
If the request fails, the strategy signals that authentication has failed.

## Examples

For a complete, working example, refer to the [SSQ signon examples](https://github.com/rivierasolutions/ssqsignon-examples) repository.

## Credits

  - [Riviera Solutions](https://github.com/rivierasolutions)

## License

[The MIT License](http://opensource.org/licenses/MIT)

Copyright (c) 2015 Riviera Solutions Piotr Wójcik <[http://rivierasoltions.pl](http://rivierasolutions.pl)>
