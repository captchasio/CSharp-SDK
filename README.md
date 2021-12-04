# C# Module for CAPTCHAs.IO API
The easiest way to quickly integrate [CAPTCHAs.IO] into your code to automate solving of any types of captcha.

- [Installation](#installation)
- [Configuration](#configuration)
- [Solve captcha](#solve-captcha)
  - [Normal Captcha](#normal-captcha)
  - [Text](#text-captcha)
  - [ReCaptcha v2](#recaptcha-v2)
  - [ReCaptcha v3](#recaptcha-v3)
  - [hCaptcha](#hcaptcha)
- [Other methods](#other-methods)
  - [send / getResult](#send--getresult)
  - [balance](#balance)
  - [report](#report)
- [Error handling](#error-handling)

## Installation
Install nuget package from [nuget]

## Configuration
`TwoCaptcha` instance can be created like this:
```csharp
TwoCaptcha solver = new TwoCaptcha('YOUR_API_KEY');
```
Also there are few options that can be configured:
```csharp
solver.SoftId = 123;
solver.Callback = "https://your.site/result-receiver";
solver.DefaultTimeout = 120;
solver.RecaptchaTimeout = 600;
solver.PollingInterval = 10;
```

### TwoCaptcha instance options

|Option|Default value|Description|
|---|---|---|
|softId|-|your software ID obtained after publishing in [CAPTCHAs.IO sofware catalog]|
|callback|-|URL of your web-sever that receives the captcha recognition result. The URl should be first registered in [pingback settings] of your account|
|defaultTimeout|120|Polling timeout in seconds for all captcha types except ReCaptcha. Defines how long the module tries to get the answer from `res.php` API endpoint|
|recaptchaTimeout|600|Polling timeout for ReCaptcha in seconds. Defines how long the module tries to get the answer from `res.php` API endpoint|
|pollingInterval|10|Interval in seconds between requests to `res.php` API endpoint, setting values less than 5 seconds is not recommended|

>  **IMPORTANT:** once `Callback` is defined for `TwoCaptcha` instance, all methods return only the captcha ID and DO NOT poll the API to get the result. The result will be sent to the callback URL.
To get the answer manually use [getResult method](#send--getresult)

## Solve captcha
When you submit any image-based captcha use can provide additional options to help CAPTCHAs.IO workers to solve it properly.

### Captcha options
|Option|Default Value|Description|
|---|---|---|
|numeric|0|Defines if captcha contains numeric or other symbols [see more info in the API docs][post options]|
|minLength|0|minimal answer lenght|
|maxLength|0|maximum answer length|
|phrase|0|defines if the answer contains multiple words or not|
|caseSensitive|0|defines if the answer is case sensitive|
|calc|0|defines captcha requires calculation|
|lang|-|defines the captcha language, see the [list of supported languages] |
|hintImg|-|an image with hint shown to workers with the captcha|
|hintText|-|hint or task text shown to workers with the captcha|

Below you can find basic examples for every captcha type. Check out [examples directory] to find more examples with all available options.

### Basic example
Example below shows a basic solver call example with error handling.

```csharp
Normal captcha = new Normal();
captcha.SetFile("path/to/captcha.jpg");
captcha.SetMinLen(4);
captcha.SetMaxLen(20);
captcha.SetCaseSensitive(true);
captcha.SetLang("en");

try
{
    await solver.Solve(captcha);
    Console.WriteLine("Captcha solved: " + captcha.Code);
}
catch (Exception e)
{
    Console.WriteLine("Error occurred: " + e.Message);
}
```

### Normal Captcha
To bypass a normal captcha (distorted text on image) use the following method. This method also can be used to recognize any text on the image.

```csharp
Normal captcha = new Normal();
captcha.SetFile("path/to/captcha.jpg");
captcha.SetNumeric(4);
captcha.SetMinLen(4);
captcha.SetMaxLen(20);
captcha.SetPhrase(true);
captcha.SetCaseSensitive(true);
captcha.SetCalc(false);
captcha.SetLang("en");
captcha.SetHintImg(new FileInfo("path/to/hint.jpg"));
captcha.SetHintText("Type red symbols only");
```

### Text Captcha
This method can be used to bypass a captcha that requires to answer a question provided in clear text.

```csharp
Text captcha = new Text();
captcha.SetText("If tomorrow is Saturday, what day is today?");
captcha.SetLang("en");
```

### ReCaptcha v2
Use this method to solve ReCaptcha V2 and obtain a token to bypass the protection.

```csharp
ReCaptcha captcha = new ReCaptcha();
captcha.SetSiteKey("6Le-wvkSVVABCPBMRTvw0Q4Muexq1bi0DJwx_mJ-");
captcha.SetUrl("https://mysite.com/page/with/recaptcha");
captcha.SetInvisible(true);
captcha.SetAction("verify");
captcha.SetProxy("HTTPS", "login:password@IP_address:PORT");
```
### ReCaptcha v3
This method provides ReCaptcha V3 solver and returns a token.

```csharp
ReCaptcha captcha = new ReCaptcha();
captcha.SetSiteKey("6Le-wvkSVVABCPBMRTvw0Q4Muexq1bi0DJwx_mJ-");
captcha.SetUrl("https://mysite.com/page/with/recaptcha");
captcha.SetVersion("v3");
captcha.SetDomain("google.com");
captcha.SetAction("verify");
captcha.SetScore(0.3);
captcha.SetProxy("HTTPS", "login:password@IP_address:PORT");
```

### hCaptcha
Use this method to solve hCaptcha challenge. Returns a token to bypass captcha.

```csharp
HCaptcha captcha = new HCaptcha();
captcha.SetSiteKey("10000000-ffff-ffff-ffff-000000000001");
captcha.SetUrl("https://www.site.com/page/");
captcha.SetProxy("HTTPS", "login:password@IP_address:PORT");
```

## Other methods

### send / getResult
These methods can be used for manual captcha submission and answer polling.

```csharp
string captchaId = await solver.Send(captcha);

Task.sleep(20 * 1000);

string code = await solver.GetResult(captchaId);
```
### balance
Use this method to get your account's balance

```csharp
double balance = await solver.Balance();
```
### report
Use this method to report good or bad captcha answer.

```csharp
await solver.Report(captcha.Id, true); // captcha solved correctly
await solver.Report(captcha.Id, false); // captcha solved incorrectly
```

## Error handling
If case of an error captcha solver throws an exception. It's important to properly handle these cases. We recommend to use `try catch` to handle exceptions.


```csharp
try
{
    await solver.Solve(captcha);
}
catch (ValidationException e)
{
    // invalid parameters passed
}
catch (NetworkException e)
{
    // network error occurred
}
catch (ApiException e)
{
    // api respond with error
}
catch (TimeoutException e)
{
    // captcha is not solved so far
}
```
