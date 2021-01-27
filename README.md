
# WorkerServiceWebAppTemplate
> A lightweight template for ASP.NET Core WebApi self-hosted in a Worker Service

A simple visual studio template for ASP.NET Core WebApi self-hosted in a Worker Service. Using this project, you can benefit worker service and web app at the same time.
Also, you needn't to install an IIS in order to run a Web API or even Web App. All you need to do is install and host the app as a windows service.

## How to
We've done following steps in order to create this simple template:
1. Create a simple console application
2. Install packages "Microsoft.AspNetCore.App" and "Microsoft.Extensions.Hosting.WindowsServices" using nuget:
	``` 
	Install-Package Microsoft.AspNetCore.App 
	```
	``` 
	Install-Package Microsoft.Extensions.Hosting.WindowsServices
	```	
	Former package is used start a self-hosted Web App / Web API in a console application, latter packages is used to host the console application as a windows service. you could install Microsoft.Extensions.Hosting.Systemd package and use it in order to host your service in a Linux server
	
3. Create a Worker.cs class library inherited from BackgroundService, it could be inherited from IHostService containing following lines of codes, this class a your background service which will be started while your start service, this task is always alive and log a message every 1 seconds.
	```
	using System;
	using System.Threading;
	using System.Threading.Tasks;
	using Microsoft.Extensions.Hosting;
	using Microsoft.Extensions.Logging;

	namespace WorkerServiceWebAppTemplate
	{
		public class Worker : BackgroundService
	    {
	        private readonly ILogger<Worker> _logger;

	        public Worker(ILogger<Worker> logger)
	        {
	            _logger = logger;
	        }

	        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
	        {
	            while (!stoppingToken.IsCancellationRequested)
	            {
	                _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
	                await Task.Delay(1000, stoppingToken);
	            }
	        }
	    }
	}
	```
4. Create a class file named Startup.cs which will initialize your web server, containing following lines of codes, it only contains a GET api at root address (/) and shows a simple message "The service is alive!".
	```
	using Microsoft.AspNetCore.Builder;
	using Microsoft.AspNetCore.Hosting;
	using Microsoft.AspNetCore.Http;
	using Microsoft.Extensions.Hosting;

	namespace WorkerServiceWebAppTemplate
	{
	    public class Startup
	    {
	        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
	        {
	            if (env.IsDevelopment())
	                app.UseDeveloperExceptionPage();
	            else
	                app.UseHsts();

	            app.UseRouting();

	            app.UseEndpoints(endpoints =>
	            {
	                endpoints.MapGet("/", async context =>
	                {
	                    await context.Response.WriteAsync("Hello World!");
	                });
	            });
	        }
	    }
	}
	```

5. Finally open your Program.cs file and add following lines of codes, the code created a HostBuilder, which will be hosted in windows services (UseWindowsService), has a worker service named Worker (ConfigureServices section) and also will start a web server with configurations set in Startup (ConfigureWebHostDefaults).
	```
	using Microsoft.AspNetCore.Hosting;
	using Microsoft.Extensions.DependencyInjection;
	using Microsoft.Extensions.Hosting;

	namespace WorkerServiceWebAppTemplate
	{
	    class Program
	    {
	        static void Main(string[] args)
	        {
	            CreateHostBuilder(args).Build().Run();
	        }

	        private static IHostBuilder CreateHostBuilder(string[] args)
	        {
	            return Host.CreateDefaultBuilder(args)
	                .UseWindowsService()
	                .ConfigureServices((hostBuilderContext, services) =>
	                {
	                    services.AddHostedService<Worker>();
	                })
	                .ConfigureWebHostDefaults(webBuilder =>
	                {
	                    webBuilder.UseStartup<Startup>();
	                });
	        }
	    }
	}
	```

## Publish settings
Publish settings should be like following screenshot
![enter image description here](https://raw.githubusercontent.com/saeedmaghdam/WorkerServiceWebAppTemplate/main/Screenshots/Publish%20Settings.JPG)

## Install as a service and test
First, publish the project using above settings. it'll produce an exe file. In our repository, it would be "WorkerServiceWebAppTemplate.exe" which will be located at "\WorkerServiceWebAppTemplate\bin\Release\netcoreapp3.1\publish\WorkerServiceWebAppTemplate.exe". 
use following command to host our service in windows services:
```
sc CREATE TestService binPath="full_path_to_exe_file"
```
And run service from windows services or using following command:
```
sc START TestService
``` 

![enter image description here](https://raw.githubusercontent.com/saeedmaghdam/WorkerServiceWebAppTemplate/main/Screenshots/Service%20installation%20and%20start%20sc.png)

After successful installation, service will be available at windows services:

![enter image description here](https://github.com/saeedmaghdam/WorkerServiceWebAppTemplate/blob/main/Screenshots/Services%20window.png?raw=true)

To test the service's Web API,  simply open url http://localhost:5000/ in your web browser and you'll see following result in your browser:

![enter image description here](https://github.com/saeedmaghdam/WorkerServiceWebAppTemplate/blob/main/Screenshots/Result.png?raw=true)


## Meta
Saeed Aghdam – [Linkedin][linkedin]

Distributed under the MIT license. See [``LICENSE``][github-license] for more information.

[https://github.com/saeedmaghdam/](https://github.com/saeedmaghdam/)

## Contributing

1. Fork it (<https://github.com/saeedmaghdam/WorkerServiceWebAppTemplate/fork>)
2. Create your feature branch (`git checkout -b feature/your-branch-name`)
3. Commit your changes (`git commit -am 'Add a short message describes new feature'`)
4. Push to the branch (`git push origin feature/your-branch-name`)

5. Create a new Pull Request

<!-- Markdown link & img dfn's -->

[linkedin]:https://www.linkedin.com/in/saeedmaghdam/
[nuget-page]:https://www.nuget.org/packages/WorkerServiceWebAppTemplate
[github]: https://github.com/saeedmaghdam/
[github-page]: https://github.com/saeedmaghdam/WorkerServiceWebAppTemplate/
[github-license]: https://raw.githubusercontent.com/saeedmaghdam/WorkerServiceWebAppTemplate/master/LICENSE