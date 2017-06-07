# 6.3 Integrating Easy Tables with Xamarin.Forms (GET Request) 

### 6.3.1 Referencing Azure Mobile Services
At the earlier sections, we would have already added it to our Nuget Packages. If not

- For Visual Studio: Right-click your project, click Manage NuGet Packages, search for the `Microsoft.Azure.Mobile.Client` package, then click Install.
- For Xamarin Studio: Right-click your project, click Add > Add NuGet Packages, search for the `Microsoft.Azure.Mobile.Client` package, and then click Add Package.

NOTE: Make sure to add it to all your solutions!

If we want to use this SDK we add the following using statement
```Csharp
using Microsoft.WindowsAzure.MobileServices;
``` 

### 6.3.2 Creating Model Classes
Lets now create model class `NotHotDogModel` to represent the tables in our database. 
So in `Moodify (Portable)`, create a folder named `DataModels` and then create a `NotHotDogModel.cs` file with,

NOTE: If your table in your backend is not called NotHotDogModel, rename it or rename this class and file to match.

```Csharp
public class NotHotDogModel
{
    [JsonProperty(PropertyName = "Id")]
    public string ID { get; set; }

    [JsonProperty(PropertyName = "Longitude")]
    public float Longitude { get; set; }

    [JsonProperty(PropertyName = "Latitude")]
    public float Latitude { get; set; }
}
``` 

#### You might have noticed that in our database we defined the "Longitude" and "Latitude" values as string but C sharp is smart enough to convert his to a float if it sees fit.  

- `JsonPropertyAttribute` is used to define the PropertyName mapping between the client type and the table 
- Important that they match the field names that we got from our postman request (else it wont map properly)
- Our field names for our client types can then be renamed if we want (like the field `date`)
- All client types must contain a field member mapped to `Id` (default a string). The `Id` is required to perform CRUD operations and for offline sync (not discussed)
 

### 6.3.3 Initalize the Azure Mobile Client
Lets now create a singleton class named `AzureManager` that will look after our interactions with our web server. Add this to the class
(NOTE: replace `MOBILE_APP_URL` with your server name, for this demo its "https://nothotdoginformation.azurewebsites.net/")


So in `Moodify (Portable)`, create a `AzureManager.cs` file with,

```Csharp
public class AzureManager
    {

        private static AzureManager instance;
        private MobileServiceClient client;

        private AzureManager()
        {
            this.client = new MobileServiceClient("MOBILE_APP_URL");
        }

        public MobileServiceClient AzureClient
        {
            get { return client; }
        }

        public static AzureManager AzureManagerInstance
        {
            get
            {
                if (instance == null) {
                    instance = new AzureManager();
                }

                return instance;
            }
        }
    }
``` 

Now if we want to access our `MobileServiceClient` in an activity we can add the following line, for the purpose of this tutorial we will be adding it to our `AzureTables.xaml.cs` file 
```Csharp
     public partial class AzureTable : ContentPage
    {
       
		MobileServiceClient client = AzureManager.AzureManagerInstance.AzureClient;

		public AzureTable()
        {
            InitializeComponent();

		}

    }
``` 


### 6.3.4 Creating a table references
For this demo we will consider a database table a `table`, so all code that accesses (READ) or modifies (CREATE, UPDATE) the table calls functions on a `MobileServiceTable` object. 
These can be obtained by calling the `GetTable` on our `MobileServiceClient` object.

Lets add our `notHotDogTable` field to our `AzureManager` activity 
```Csharp
    private IMobileServiceTable<NotHotDogModel> notHotDogTable;
``` 

And then the following line at the end of our `private AzureManager()` function
```Csharp
    this.notHotDogTable = this.client.GetTable<NotHotDogModel>();
```

This grabs a reference to the data in our `NotHotDogModel` table in our backend and maps it to our client side model defined earlier.

We can then use this table to actually get data, get filtered data, get a NotHotDogModel by id, create new NotHotDogModel, edit NotHotDogModel and much more.

### 6.3.5 Grabbing NotHotDogModel data
To retrieve information about the table, we can invoke a `ToListAsync()` method call, this is asynchronous and allows us to do LINQ querys.

Lets create a `GetHotDogInformation` method in our `AzureManager.cs` file
```Csharp
    public async Task<List<NotHotDogModel>> GetHotDogInformation() {
        return await this.notHotDogTable.ToListAsync();
    }
``` 
#### Your AzureManager.cs Class should like like the code snippet below now
```Csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.MobileServices;

namespace Tabs
{
	public class AzureManager
	{

		private static AzureManager instance;
		private MobileServiceClient client;
		private IMobileServiceTable<NotHotDogModel> notHotDogTable;

		private AzureManager()
		{
			this.client = new MobileServiceClient("https://nothotdoginformation.azurewebsites.net");
            this.notHotDogTable = this.client.GetTable<NotHotDogModel>();
		}

		public MobileServiceClient AzureClient
		{
			get { return client; }
		}

		public static AzureManager AzureManagerInstance
		{
			get
			{
				if (instance == null)
				{
					instance = new AzureManager();
				}

				return instance;
			}
		}

		public async Task<List<NotHotDogModel>> GetHotDogInformation()
		{
			return await this.notHotDogTable.ToListAsync();
		}
	}
}
```

Add the following to your `AzureTable.xaml` 

```xml
  <?xml version="1.0" encoding="UTF-8"?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms" xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml" x:Class="Tabs.AzureTable" Title="Information">
    <ContentPage.Padding>
        <OnPlatform x:TypeArguments="Thickness" iOS="0, 20, 0, 0" />
    </ContentPage.Padding>
    <ContentPage.Content>
        <StackLayout>
            <Button Text="See Photo Information" TextColor="White" BackgroundColor="Red" Clicked="Handle_ClickedAsync" />
            <ListView x:Name="HotDogList" HasUnevenRows="True">
                <ListView.ItemTemplate>
                    <DataTemplate>
                        <ViewCell>
                            <StackLayout Orientation="Horizontal">
                                <Label Text="{Binding Longitude, StringFormat='Longitude: {0:N}'}" HorizontalOptions="FillAndExpand" Margin="20,0,0,0" VerticalTextAlignment="Center" />
                                <Label Text="{Binding Latitude, StringFormat='Latitude: {0:N}'}" VerticalTextAlignment="Center" Margin="0,0,20,0" />
                            </StackLayout>
                        </ViewCell>
                    </DataTemplate>
                </ListView.ItemTemplate>
            </ListView>
        </StackLayout>
    </ContentPage.Content>
</ContentPage>
```
 - Here we added a template for the NotHotDogModel object values, showing the `Longitude`, and `Latitude` values by using `Binding` ie `"{Binding Longitude, StringFormat='Longitude: {0:N}'}"`. This is a very simple way to display all our values and can be futher extended to display it in a aesthetic manner.
This associates the value of the field of the NotHotDogModel object and displays it. The string formatting used is conventional C# syntax. 
- Since The Longitude and Latitude values are of type float the 0:N syntax will automatically round the information to two decimal places, unless specified otherwise. 


Now to can call our `GetHotDogInformation` function, we can add the following method in our `AzureTables.xaml.cs` class
```Csharp
    
    async void Handle_ClickedAsync(object sender, System.EventArgs e)
    {
        List<NotHotDogModel> notHotDogInformation = await AzureManager.AzureManagerInstance.GetHotDogInformation();

        HotDogList.ItemsSource = notHotDogInformation;
    }
``` 

This will then set the source of the list view  `NotHotDogModel` to the list of NotHotDogModel information we got from our backend.

Your `AzureTables.xaml.cs` class should now look like this: 

```Csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.MobileServices;
using Xamarin.Forms;

namespace Tabs
{
    public partial class AzureTable : ContentPage
    {
       
		MobileServiceClient client = AzureManager.AzureManagerInstance.AzureClient;

		public AzureTable()
        {
            InitializeComponent();

		}

		async void Handle_ClickedAsync(object sender, System.EventArgs e)
		{
			List<NotHotDogModel> notHotDogInformation = await AzureManager.AzureManagerInstance.GetHotDogInformation();

			HotDogList.ItemsSource = notHotDogInformation;
		}

    }
}
```

[More Info on ListView](https://developer.xamarin.com/guides/xamarin-forms/user-interface/listview/) about customising the appearance of your list view


#### `[Extra Information]` 
##### *Example doesn't apply in this tutorial, just information to let you know that this is an option available*
A LINQ query we may want to achieve is if we want to filter the data to only return high happiness songs. 
We could do this by the following line, this grabs the NotHotDogModel information if it has a happiness of 0.5 or higher
```Csharp
    public async Task<List<NotHotDogModel>> GetHotDogInformation() {
        return await notHotDogTable.Where(notHotDogInformation => notHotDogInformation.Happiness > 0.5).ToListAsync();
    }
``` 

### Extra Learning Resources
* [Using App Service with Xamarin by Microsoft](https://azure.microsoft.com/en-us/documentation/articles/app-service-mobile-dotnet-how-to-use-client-library/)
* [Using App Service with Xamarin by Xamarin - Outdated but good to understand](https://blog.xamarin.com/getting-started-azure-mobile-apps-easy-tables/)
* [ListView in Xamarin](https://developer.xamarin.com/guides/xamarin-forms/user-interface/listview/)