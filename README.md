Initially MonkeyFinder App has blank code with view,model,resource,service,platform,viewmodel
![Screenshot 2023-09-25 133450](https://github.com/Wichita-State-CS797R/cs797r-project1-workshop-ramya-2701/assets/95271051/47af5b08-07b5-40d3-86a2-faa9cd0df388)

Model/Monkey.cs:
We have added the c# code to get json data about monkeys.Additionally, because we will be using System.Text.Json to deserialize the data, we will want to add a MonkeyContext that will dynamically generate code for better performance.We are adding the monkey details in the collection view tag in the MainPage.Xaml.
Debug the code Debug->start debugging
Result:
![Screenshot 2023-09-24 223656](https://github.com/Wichita-State-CS797R/cs797r-project1-workshop-ramya-2701/assets/95271051/4f27c37e-1ba8-4796-b5df-ee28fd28e510)

To display the strings vertically like a stack add the following code in MainPage.Xaml:

 <HorizontalStackLayout Padding="10">
    <Image
        Aspect="AspectFill"
        HeightRequest="100"
        Source="{Binding Image}"
        WidthRequest="100" />
    <VerticalStackLayout VerticalOptions="Center">
        <Label Text="{Binding Name}" FontSize="24" TextColor="Gray"/>
        <Label Text="{Binding Location}" FontSize="18" TextColor="Gray"/>
    </VerticalStackLayout>
</HorizontalStackLayout>

Result:

![Screenshot 2023-09-25 220948](https://github.com/Wichita-State-CS797R/cs797r-project1-workshop-ramya-2701/assets/95271051/84af8ea6-08fa-4c29-b29c-b0f64f0c0e7e)


Creating properties like Title,IsBusy,IsNotBusy in BasicViewModel.cs. These properties allow us to not to perform duplicate operations when our viewModel is busy and also allow us to set the title on our pages.OnPropertyChanged() method is called when the value changes. We can implement all these properties simply with thw ObservableProperty as shown below.
![Screenshot 2023-09-27 125335](https://github.com/Wichita-State-CS797R/cs797r-project1-workshop-ramya-2701/assets/95271051/96b1b320-a06c-48f9-9d2e-365b23774b91)

Have to create GetMonkeys() method in MonkeyService.cs file to retrive monkeys data from internet using HTTP request to the client. To display all the extracted monkeys in the App we have to call MonkeyService from MonkeysViewModel.We use an ObservableCollection because it has built-in support to raise CollectionChanged events when we Add or Remove items from the collection.Created the GetMonkeysAsync() method with try/catch/finally blocks to overcome the exceptions that returns async tasks.Next step is to register the MainPage with builder.Services.Build Monkeys user interface by adding code in the MainPage.Xaml.

<ContentPage
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    x:Class="MonkeyFinder.View.MainPage"
    xmlns:model="clr-namespace:MonkeyFinder.Model"
    xmlns:viewmodel="clr-namespace:MonkeyFinder.ViewModel"
    x:DataType="viewmodel:MonkeysViewModel"
    Title="{Binding Title}">
   <Grid
        ColumnDefinitions="*,*"
        ColumnSpacing="5"
        RowDefinitions="*,Auto"
        RowSpacing="0">
        <CollectionView ItemsSource="{Binding Monkeys}"
                         SelectionMode="None"
                         Grid.ColumnSpan="2">
            <CollectionView.ItemTemplate>
                <DataTemplate x:DataType="model:Monkey">
                    <Grid Padding="10">
                        <Frame HeightRequest="125" Style="{StaticResource CardView}">
                            <Grid Padding="0" ColumnDefinitions="125,*">
                                <Image Aspect="AspectFill" Source="{Binding Image}"
                                       WidthRequest="125"
                                       HeightRequest="125"/>
                                <VerticalStackLayout
                                    Grid.Column="1"
                                    VerticalOptions="Center"
                                    Padding="10">
                                    <Label Style="{StaticResource LargeLabel}" Text="{Binding Name}" />
                                    <Label Style="{StaticResource MediumLabel}" Text="{Binding Location}" />
                                </VerticalStackLayout>
                            </Grid>
                        </Frame>
                    </Grid>
                </DataTemplate>
            </CollectionView.ItemTemplate>
        </CollectionView>
        <Button Text="Get Monkeys" 
                Command="{Binding GetMonkeysCommand}"
                IsEnabled="{Binding IsNotBusy}"
                Grid.Row="1"
                Grid.Column="0"
                Style="{StaticResource ButtonOutline}"
                Margin="8"/>
        <ActivityIndicator IsVisible="{Binding IsBusy}"
                           IsRunning="{Binding IsBusy}"
                           HorizontalOptions="Fill"
                           VerticalOptions="Center"
			   Color="{StaticResource Primary}"
                           Grid.RowSpan="2"
                           Grid.ColumnSpan="2"/>
    </Grid>
</ContentPage>
Result:
We get the list of monkeys when we hit the Monkeys List Button as shown in the screenshot
![Screenshot 2023-09-25 235320](https://github.com/Wichita-State-CS797R/cs797r-project1-workshop-ramya-2701/assets/95271051/e4707256-0189-42cf-972e-c56c14169f20)

Now add the navigation to the Monkeys List page which will redirect to the monkey details page. Create a method async Task GoToDetailsAsync(Monkey monkey) exposed as an [RelayCommand] in MonkeysViewModel.cs, This will check the item is not null and navigates to the details page which contains the monkey details.
Add the following code in the MonkeysDetailsViewModel to create the bindable properties:

//Add QueryProperty
[QueryProperty(nameof(Monkey), "Monkey")]
public partial class MonkeyDetailsViewModel : BaseViewModel
{
    public MonkeyDetailsViewModel()
    {
    }

    [ObservableProperty]
    Monkey monkey;
}


In AppShell.xaml.cs add the following code for routing:

Routing.RegisterRoute(nameof(DetailsPage), typeof(DetailsPage));

In mauiProgram.cs add the builderServices:

builder.Services.AddTransient<MonkeyDetailsViewModel>();
builder.Services.AddTransient<DetailsPage>();

Create user interface for Details page by adding grid,label,verticalStackLayout to the page for displaying it.Run the code it is displayed as shown:

![Screenshot 2023-09-26 004119](https://github.com/Wichita-State-CS797R/cs797r-project1-workshop-ramya-2701/assets/95271051/f83031a8-6491-41d9-a9ef-b19363ce5a53)


Inject IConnectivity in the MonkeysViewModel constructor.Register Connectivity.Current,Geolocation.Default,Map.Default in MauiProgram.cs and then add the following code in GetMonkeyAsync method to check the internet connection:

if (connectivity.NetworkAccess != NetworkAccess.Internet)
{
    await Shell.Current.DisplayAlert("No connectivity!",
        $"Please check internet and try again.", "OK");
    return;
}

Turnoff the wifi network and run the code then it display as shown:

![Screenshot 2023-09-27 104026](https://github.com/Wichita-State-CS797R/cs797r-project1-workshop-ramya-2701/assets/95271051/13abd8a8-c513-4452-b0de-f8bb477a22f9)

Add IGeolocator in the MonkeysViewModel to add the functionality of GPS. To find the closest monkey add the GetClosestMonkey method in MonkeysViewModel.cs and also add the Find Closest Monkey button in the MainPage.Xaml. Run the code when we click on the find closest it displays the closest monkey as shown:

![Screenshot 2023-09-26 211951](https://github.com/Wichita-State-CS797R/cs797r-project1-workshop-ramya-2701/assets/95271051/89c77271-1de2-4b63-85f0-1d19cfe187cf)

Add shown on Map feature for every monkey in the details page by adding the Show on Map button in the DetailsPage.Xaml. Also inject IMap in MonkeysViewModel and also add the openMap method which the Map API passing it the monkey's location.

[RelayCommand]
async Task OpenMap()
{
    try
    {
        await map.OpenAsync(Monkey.Latitude, Monkey.Longitude, new MapLaunchOptions
        {
            Name = Monkey.Name,
            NavigationMode = NavigationMode.None
        });
    }
    catch (Exception ex)
    {
        Debug.WriteLine($"Unable to launch maps: {ex.Message}");
        await Shell.Current.DisplayAlert("Error, no Maps app!", ex.Message, "OK");
    }
}

Re-run the code to see the results:

![Screenshot 2023-09-26 220650](https://github.com/Wichita-State-CS797R/cs797r-project1-workshop-ramya-2701/assets/95271051/862ef5b9-80de-4cd2-a7c9-e26c6c5ed5a2)

when we click on the MonkeyLocation button it will be redirected to the location of monkey in the maps.

Add RefreshView to add pull to refresh in the collectionView:

<RefreshView
    Grid.ColumnSpan="2"
    Command="{Binding GetMonkeysCommand}"
    IsRefreshing="{Binding IsRefreshing}">
    <ContentView>
        <CollectionView
            ItemsSource="{Binding Monkeys}"
            SelectionMode="None">
            <!-- Template -->
        </CollectionView>
    </ContentView>
</RefreshView>

open MonkeysViewModel.cs and add the following property and edit finally block by adding  IsRefreshing = false:

[ObservableProperty]
bool isRefreshing;

Run the code and pull the page to see the result as shown in ss:

![Screenshot 2023-09-26 225538](https://github.com/Wichita-State-CS797R/cs797r-project1-workshop-ramya-2701/assets/95271051/454cfb6b-e611-4b04-9bb9-c57811ab2c97)

We have implemented linear layout, grid layout, horizontal layout and empty view by adding the CollectionView.ItemsLayout.

Linear Layout:
![Screenshot 2023-09-27 003901](https://github.com/Wichita-State-CS797R/cs797r-project1-workshop-ramya-2701/assets/95271051/72f3aa0c-77a6-4a1c-b4de-ea2937745a82)

grid layout:

![Screenshot 2023-09-26 231212](https://github.com/Wichita-State-CS797R/cs797r-project1-workshop-ramya-2701/assets/95271051/2cefc878-cf67-4723-9654-d151f486f4df)

horizontal layout:

![Screenshot 2023-09-27 112530](https://github.com/Wichita-State-CS797R/cs797r-project1-workshop-ramya-2701/assets/95271051/b386fe13-355f-4fb9-b449-bfada8752f1a)


Empty View:

![Screenshot 2023-09-26 232743](https://github.com/Wichita-State-CS797R/cs797r-project1-workshop-ramya-2701/assets/95271051/8bd2a634-449d-410d-b9fa-71a2760d3c44)





