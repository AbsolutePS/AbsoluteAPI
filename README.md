
# AbsoluteAPI
Absolute Public API library for .Net. This library is available on Nuget and is provided and supported by the Absolute Professional Services team.

## Get from Nuget
Nuget package is available here:
https://www.nuget.org/packages/AbsolutePublicAPI/2.0.0



## Getting Started
To get started, you need to get an access token from the Absolute console. Depending on what data center you are located, you can get it from https://cc.absolute.com/ (Canadian DC) or https://cc.us.absolute.com/ (United States DC). Detailed instructions on getting the API Tokens required for access can be found at [https://www.absolute.com/media/2221/abt-api-working-with-absolute.pdf](https://www.absolute.com/media/2221/abt-api-working-with-absolute.pdf) .

Once you have an API token, you can get started with configuration for the AbsoluteAPI class.

### AbsoluteAPI Configuration
There are a few different options for configuration the AbsoluteAPI class.

#### IMPORTANT: Account Region
Absolute has multiple data centers and the configuration settings accounts for this. In the examples below, the CADC (Canadian Data Center) is used by default (https://cc.absolute.com users will use https://api.absolute.com). If you have an alternative data center, you need its DefaultDomain and Region values. For example, the USDC (United States Data Center) is "api.us.absolute.com" for DefaultDomain and "usdc" for "Region".

If you login to a different URL that cc.absolute.com or cc.us.absolute.com, contact Absolute Support for details on how to access your region via the Public API and what domain and region values apply to you.

#### Configuration by File
You can specify a configuration file to be used by the library at runtime. It will read and use the contents.
To use this, use a constructor overload that includes the configuration file name. Note that if the file path is not provided, it automatically assumes "appsettings.json" which will work in .Net Core (if the file is present in the project) but not in .Net Framework. If using .Net Framework, we recommend explicitly using ".\\appsettings.json" as the relative path value.

    // Works in .Net Core for relative path
    var api = new AbsoluteAPI(
            "TOKEN",
            "SECRET",
            "api.absolute.com",
            "appsettings.json");
	    
    // Works in .Net Framework for relative path
    var api = new AbsoluteAPI(
            "TOKEN",
            "SECRET",
            "api.absolute.com",
            ".\\appsettings.json");
	   
File contents:

    {
      "DefaultDomain": "api.absolute.com",
      "ApiVersion": "v2",
      "Service": "abs1",
      "Region":  "cadc"
    }

#### Configuration by Code
You can also provide the configuration and token via an AbsoluteAPI constructor so you can load or use hard-coded settings if you like, and your application needs no configuration file at all. You'll notice that the content is the same as the above, and the DefaultDomain

First, create an AbsoluteAPIConfiguration object:

    var configuration = new AbsoluteAPIConfiguration()
                {
                    DefaultDomain = "api.absolute.com",
                    ApiVersion = "v2",
                    Region = "cadc",
                    Service = "abs1"
                };
Then, create an AbsoluteAPIToken object:

    var token = new AbsoluteAPIToken()
                {
                    Name = "Token Name",
                    Token = "TOKEN",
                    Secret = "SECRET"
                };
Finally, create the AbsoluteAPI object:

    var api = new AbsoluteAPI(configuration, token);

### Request Configuration
The RequestConfiguration class is a convenient way to configure requests and utilizies a fluent API to allow you to quickly modify request configuration without creating many objects of the same type.

#### Fluent API Options
The following fluent API methods are available for this class:

 - WithTop(int top)
	 - Limits number of results for each request
	 - Default value of 1000 if not specified
 - WithSkip(int skip)
	 - Skips this number of records (for use with paging/iterating through results)
	 - Default value of 0 if not specified
 - WithOrderBy(string orderBy)
	 - Orders results. Use OData standard string for orderBy and can be used for desc as well, e.g. "lastConnectedUtc desc" is valid orderby value.
	 - Default value is empty string
 - WithFilter(string filter)
	 - For filtering results. Use OData standard string for filter, e.g. "agentStatus eq 'A'" is a valid string.
 - WithSelect(string select)
	 - For selecting only specific fields of results. Use OData standard string for select, e.g. "esn,cdf" to select just the ESN and CDF fields of a device.

#### Examples
Here are some examples to get you started:

	    // Default values
	    var requestConfig = new RequestConfiguration();
	    
	    // With a basic filter for single device
    	var requestConfig = new RequestConfiguration()
                        .WithFilter("esn eq '2DE05RUEG0AA1V7S0003'");
        
        // With a basic filter and select values
        var requestConfig = new RequestConfiguration()
                .WithSelect("esn,id")
                .WithTop(100);
        
        // With a more complex filter containing substringof OData method
    	var requestConfig = new RequestConfiguration()                    
                        .WithFilter("substringof('0003', esn) eq true");
        
        // More complex request with multiple filters, limiting to devices connected more recently than specific time and with agent status of Active
    	var requestConfig = new RequestConfiguration()
                    .WithFilter("((agentStatus eq 'A') and (lastConnectedUtc ge datetime'2000-01-01T00:00:00Z'))")
                    .WithSkip(0)
                    .WithTop(400);


## Supported API Endpoints
Now that you have configuration options ready to go, you can easily make requests to get device data from the Reporting Devices API.

Each of the API endpoints supported has its own methods for getting the data from the AbsoluteAPI class. ReportingDevices endpoint is the most basic of those and is discussed first.

To allow for the dynamic nature of device data, each supported endpoint generally has two return types: A basic, pre-defined class (that can be inherited to add more fields as desired), or a dynamic return type that can be used via the dynamic class type in .Net. It's worth noting that users should take extra care using dynamic: runtime errors are not uncommon and can be hard to handle all edge cases with dynamic types.

### Reporting Devices Endpoint
This endpoint is used for getting device data from Absolute. This is the most common API endpoint to use.

#### ReportingDevice Return
The method GetDevices<T>(RequestConfiguration configuration) is available but must use either the ReportingDevice class as the return type (the default, pre-defined class) or a class that inherits from ReportingDevice (if you want to add fields not included or not available at the time the ReportingDevice class was defined).

Example of this method:

    var requestConfig = new RequestConfiguration();
    
    var devices = await _api.ReportingDevices.GetDevices<ReportingDevice>(requestConfig);



##### Inheriting from ReportingDevice
We can inherit from ReportingDevice to get all the default fields and an additional field if it wasn't present on the ReportingDevice class:

 

    class TestClass : ReportingDevice
            {
                public string NewField { get; set; }
            }

The inherited class can be used like before with the custom type instead, as seen below with GetDevices\<TestClass\>:

    var requestConfig = new RequestConfiguration();
    var devices = await _api.ReportingDevices.GetDevices<TestClass>(requestConfig);

#### Dynamic return
The method GetDevicesDynamic(RequestConfiguration configuration) is available but the caller must accept return type of dynamic. This can be easier or harder to deal with depending on your familiarity with the dynamic features in C#.

Example of this method:

    var requestConfig = new RequestConfiguration()
                    .WithSelect("esn,id")
                    .WithTop(100);
		    
    var devices = await _api.ReportingDevices.GetDevicesDynamic(requestConfig);

##### Obligatory warning about "dynamic"
Note that the "devices" in the dynamic return case is a List\<dynamic\> and any field is accessible at this point (dynamically at runtime). **However, be very careful**: if you try to access a field that does not exist, there will be a runtime exception thrown by your application.


### Custom Device Fields Endpoint

This endpoint is used for getting device data from Absolute. This is the most common API endpoint to use.

#### CustomDeviceFields Return
The method GetForDevice<T>(string deviceUid, RequestConfiguration configuration) is available but must use either the CustomDeviceFields class as the return type (the default, pre-defined class) or a class that inherits from CustomDeviceFields in case you want to use a custom class for the return type.

Example of this method:

    var requestConfig = new RequestConfiguration();
    var devices = await _api.ReportingDevices.GetDevices<ReportingDevice>(requestConfig);
    CustomDeviceFields fields = null;
    foreach(var device in devices)
    {
    	var deviceUid = device.Id;                    
    	fields = await _api.CustomDeviceFields.GetForDevice<CustomDeviceFields>((string)deviceUid, requestConfig);
    }

##### Inheriting from ReportingDevice
We can inherit from CustomDeviceFields to get all the default fields and an additional field if it wasn't present on the CustomDeviceFields class:

    class TestClass : CustomDeviceFields
            {
                public string NewField { get; set; }
            }

The inherited class can be used just like in previous example of code but with the custom type instead, as seen below with GetForDevice\<TestClass \>:

    fields = await _api.CustomDeviceFields.GetForDevice<TestClass>(deviceUid, requestConfig);

#### Dynamic return
The method GetForDeviceDynamic(string deviceUid, RequestConfiguration configuration) is available but the caller must accept return type of dynamic. This can be easier or harder to deal with depending on your familiarity with the dynamic features in C#.

Example of this method:

    var requestConfig = new RequestConfiguration();
    var devices = await _api.ReportingDevices.GetDevices<ReportingDevice>(requestConfig);
    dynamic fields = null;
    foreach(var device in devices)
    {
    	var deviceUid = device.Id;                    
    	fields = await _api.CustomDeviceFields.GetForDeviceDynamic<CustomDeviceFields>((string)deviceUid, requestConfig);
    }

##### Obligatory warning about "dynamic"
Note that the "devices" in the dynamic return case is a List\<dynamic\> and any field is accessible at this point (dynamically at runtime). **However, be very careful**: if you try to access a field that does not exist, there will be a runtime exception thrown by your application.
