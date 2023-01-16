# Implementing the Mobile Application

The particularity of XAML compared to Windows Forms is that the graphical user interface (GUI) must also be implemented in code. The code that defines the content of the initial page (or Log In page) can be seen below:

```xml
<?xml version="1.0" encoding="utf-8" ?>

<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"

             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"

             x:Class="csNEA_mobileApp.LoginScreen"

             BackgroundColor="{AppThemeBinding Dark=Black, Light=White}">

    <ContentPage.Content>

        <StackLayout HorizontalOptions="Center" VerticalOptions="Center">

            <Label Text="Welcome! Please log in."/>

            <Entry x:Name="entUserName" Placeholder="Username" />

            <Entry x:Name="entPasswd" Placeholder="Password" IsPassword="True" />

            <Entry x:Name="entAddress" Placeholder="Database Address" />

            <Entry x:Name="entDBPass" Placeholder="Database Password" />

            <Button Text="Log In" Clicked="Button_Clicked"/>

        </StackLayout>

    </ContentPage.Content>

</ContentPage>
```

Similarly implemented as HTML code, objects are defined and properties such as their name, Clicked event name or text can be set. The name is especially useful to be able to link to it from the code-behind as seen below:

```csharp
using System;

using System.Collections.Generic;

using System.Data.SqlClient;

using System.Linq;

using System.Text;

using System.Threading.Tasks;

using Xamarin.Forms;

using Xamarin.Forms.Xaml;

namespace csNEA_mobileApp

{

    [XamlCompilation(XamlCompilationOptions.Compile)]

    public partial class LoginScreen : ContentPage

    {       

        List<User> listOfUsers = new List<User>() { }; //users retrieved from the database will be stored here

        User tempUser;

        public LoginScreen()

        {          

            InitializeComponent();

            if (Settings.CurrentDatabase != String.Empty) //enables database information to be retrieved after the user has logged out

            {

                entAddress.Text = Settings.CurrentDatabase;

                entDBPass.Text = Settings.CurrentDBPassword;

            }

        }

        private void Button_Clicked(object sender, EventArgs e)

        {

            bool success = false;

            SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder(); //where database info is temporarily stored

            builder.DataSource = entAddress.Text;

            builder.UserID = "adminDB";

            builder.Password = entDBPass.Text;

            builder.InitialCatalog = "aradippou5";

            Settings.CurrentDatabase = entAddress.Text;

            Settings.CurrentDBPassword = entDBPass.Text; //remembering the database information for future app launches

            try

            {

                using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

                {

                    String sql = "SELECT UserName, UserPassword, FirstName FROM dbo.Users WHERE UserRole='t';"; //Selecting Teachers Only

                    using (SqlCommand command = new SqlCommand(sql, connection))

                    {

                        connection.Open();

                        using (SqlDataReader reader = command.ExecuteReader())

                        {

                            while (reader.Read())

                            {

                                AddUser(reader.GetString(0), reader.GetString(1), reader.GetString(2));

                            }

                            reader.Close();

                        }

                        connection.Close();

                    }

                }

                for (int i = 0; i < listOfUsers.Count; i++)

                {

                    if (entUserName.Text == listOfUsers[i].username && entPasswd.Text == listOfUsers[i].password)

                    {

                        success = true; //input has been matched with a user's credentials

                        Settings.CurrentUsername = listOfUsers[i].username; //remembering the user's username for future launches

                        MainPage.SetDBinfo(); //sends the database information to the main page

                        String sql = "UPDATE dbo.Users SET IsLoggedIn = 1 WHERE UserName = '" + listOfUsers[i].username + "';"; //will set the user as being logged in

                        using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

                        {

                            using (SqlCommand command = new SqlCommand(sql, connection))

                            {

                                connection.Open();

                                command.ExecuteNonQuery();

                                connection.Close();

                            }

                        }

                        listOfUsers.Clear();

                        Settings.FirstRun = false;

                        break;

                    }

                }

                if (success == false)

                {

                    ShowMessage(1);

                    listOfUsers.Clear();

                }

                else //success == true

                    this.Navigation.PopModalAsync();

            }

            catch

            {

                ShowMessage(2);

            }

        }

        private void AddUser(string username, string password, string firstName)

        {

            tempUser = new User

            {

                username = username,

                password = password,

                firstName = firstName

            };

            listOfUsers.Add(tempUser);

        }

        async void ShowMessage(int reason)

        {

            if (reason == 1) //the database info is correct but user does not exist or username and/or password is incorrect

                await DisplayAlert("Alert", "The Username and/or Password are incorrect.", "OK");

            else //the database could not be reached in the first place

                await DisplayAlert("Alert", "Could not contact the Database.", "OK");

        }       

    }

    public class User

    {

        public string username, password, firstName;

    }
}
```

After logging in successfully, the Log In page will be popped and the main page will be on display. The code of the main page is run first regardless, since it is the first page to be accessed in the namespace and thus added to the stack of pages. The Log In page will be pushed onto the stack in case there is no user currently signed into the application. The stack is managed by Xamarin Forms.

Designing the Main Page is done with the below XAML code:

```xml
<?xml version="1.0" encoding="utf-8" ?>

<TabbedPage xmlns="http://xamarin.com/schemas/2014/forms"

             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"

             x:Class="csNEA_mobileApp.MainPage">

    <ContentPage x:Name="attendancePage" Title="Attendance" IconImageSource="baseline_assignment_black_36dp.png" BackgroundColor="{AppThemeBinding Dark=Black, Light=White}">

        <ContentPage.Content>

            <StackLayout>

                <Frame BackgroundColor="#2196F3" Padding="34" CornerRadius="4">

                    <Label x:Name="frameMsg" HorizontalTextAlignment="Center" VerticalOptions="EndAndExpand" TextColor="White" FontSize="24" />

                </Frame>

                <Label Text="Want to fill in the register?" FontSize="18" VerticalOptions="End" TextColor="{AppThemeBinding Dark=White, Light=Black}"/>

                <Label Text="Select a period from the list and press the button below to take attendances:" FontSize="16" Padding="30,0,30,0" TextColor="{AppThemeBinding Dark=White, Light=Black}" />

                <Picker x:Name="pickerGroup" Title="Select a Period" />

                <Button x:Name="btnTakeAttendance" Text="Take Attendance" BorderWidth="1" Clicked="BtnTakeAttendance_Clicked"/>

            </StackLayout>

        </ContentPage.Content>

    </ContentPage>

    <?the above defines the Attendance page, the first of the three pages in the TabbedPage?>

    <ContentPage Title="Feed" IconImageSource="baseline_view_day_black_36dp.png">

        <ListView x:Name="lstFeed"

                  ItemsSource="{Binding Path=BindingContext.posts}"

                  IsVisible="True"

                  HasUnevenRows="True"

                  SeparatorColor ="Aqua"

                  IsPullToRefreshEnabled="True"

                  RefreshCommand="{Binding RefreshCommand}"

                  IsRefreshing="{Binding IsRefreshing}">

            <?note that data binding here binds the SOURCE of the contents of the listView to an ObservableCollection in c# along with the algorithm to refresh the list?>

            <ListView.Header>

                <StackLayout>

                    <Frame BackgroundColor="ForestGreen" Padding="24" CornerRadius="4">

                        <Label Text="Feed" HorizontalOptions="Center" VerticalOptions="EndAndExpand" TextColor="White" FontSize="24" />

                    </Frame>

                </StackLayout>

            </ListView.Header>

            <ListView.ItemTemplate>

                <DataTemplate>

                    <ViewCell>

                        <StackLayout Orientation="Vertical"> <?the three lines below define a list member, and are bound to the data type FeedPost?>

                            <Label Text="{Binding User}" FontSize="Title"></Label>

                            <Label Text="{Binding DatePosted}" FontSize="Subtitle"></Label>

                            <Label Text="{Binding PostText}" FontSize="Body"></Label>

                        </StackLayout>

                    </ViewCell>

                </DataTemplate>

            </ListView.ItemTemplate>

        </ListView>

    </ContentPage>

    <?the above defines the Feed page, the second of the three pages in the TabbedPage?>

    <ContentPage Title="Settings" IconImageSource="baseline_settings_black_36dp.png" BackgroundColor="{AppThemeBinding Dark=Black, Light=White}">

        <StackLayout HorizontalOptions="Center" VerticalOptions="Center">

            <Frame BackgroundColor="Orange" Padding="24" CornerRadius="4" VerticalOptions="Start">

                <Label Text="Settings" HorizontalOptions="Center" VerticalOptions="EndAndExpand" TextColor="White" FontSize="24" />

            </Frame>

            <Label Text="Change Password:" FontSize="20" TextColor="{AppThemeBinding Dark=White, Light=Black}" />

            <Entry x:Name="entCurrentPasswd" Placeholder="Current Password" IsPassword="True" />

            <Entry x:Name="entNewPasswd" Placeholder="New Password" IsPassword="True" />

            <Entry x:Name="entConfirmPasswd" Placeholder="Confirm New Password" IsPassword="True" />

            <Button Text="Submit Changes" Clicked="BtnChangePasswd_Clicked" />

            <Label HorizontalOptions="Center" Text="To Log Out altogether, press the button below:" FontSize="18" TextColor="{AppThemeBinding Dark=White, Light=Black}" />

            <Button TextColor="Red" Text="Log Out" Clicked="BtnLogOut_Clicked" />

        </StackLayout>

    </ContentPage>

    <?the above defines the Settings page, the third and final page in the TabbedPage structure?>

</TabbedPage>
```

Data binding is used here, since the members of the ListView in the Feed page are constructed through the FeedPost class, the code of which can be found below:

```csharp
using System;

using System.Collections.Generic;

using System.Text;

namespace csNEA_mobileApp

{

    public class FeedPost

    {

        public FeedPost(string postText, string user, DateTime datePosted)

        {

            PostText = postText;

            User = user;

            DatePosted = datePosted;

        }

        public string PostText { get; set; }

        public DateTime DatePosted { get; set; }

        public string User { get; set; }

    }
}
```

As a list of the type ‘FeedPost’ is updated while the program is running, the contents of that list will be dynamically shown in the ListView due to this bond. Additionally, the ‘refresh’ state of the ListView is bound to a subroutine in the code-behind, to remove all members of the list and get them again from the database.

The c# implementation of the Main Page, or the ‘code-behind’ can be found below:

```csharp
using System;

using System.Collections.Generic;

using System.Collections.ObjectModel;

using System.ComponentModel;

using System.Data.SqlClient;

using System.Diagnostics;

using System.Linq;

using System.Text;

using System.Threading.Tasks;

using System.Windows.Input;

using Xamarin.Forms;

namespace csNEA_mobileApp

{

    public partial class MainPage : TabbedPage

    {

        public ObservableCollection<FeedPost> posts { get; set; } //similar to a list, will hold the collection of feed posts to be displayed

        public static SqlConnectionStringBuilder builder { get; set; } //holds the database information

        private bool _isRefreshing = false;

        public static List<int> CurrentPeriods { get; set; } //holds the periods of the specific day according to the logged in user's timetable

        public bool logoutConfrm { get; set; }

        public bool IsRefreshing //determins whether the status of the ListView in the GUI is 'refreshing'

        {

            get { return _isRefreshing; }

            set

            {

                _isRefreshing = value;

                OnPropertyChanged(nameof(IsRefreshing));

            }

        }

        public ICommand RefreshCommand //fires when the user causes the ListView in the GUI to go into the 'refreshing' state

        {

            get

            {

                return new Command(() =>

                {

                    IsRefreshing = true;

                    UpdateFeed();

                    IsRefreshing = false;

                });

            }

        }

        public MainPage()

        {

            InitializeComponent();

            if (Settings.FirstRun == true)

            {

                ShowLogin();               

            }

            else

            {

                SetDBinfo();

                UpdateFrame();

                try

                {

                    UpdateFeed();

                    UpdatePeriods();

                }

                catch

                {

                    ShowErrorMessage(1);

                }

            }

            //DateTime localDate = DateTime.Now;

            this.BindingContext = this;

        }
```

The above code is run first, which will determine whether a user has logged in previously or if the application was run without a user signed in. For the first condition, their timetable and feed posts are retrieved from the database. Otherwise the log in page will be pushed onto the stack of pages managed by Xamarin Forms with the procedure ShowLogin( ), shown below:

```csharp
        public async void ShowLogin()

        {

            LoginScreen modalPage = new LoginScreen();

            modalPage.Disappearing += (sender2, e2) =>

            {

                System.Diagnostics.Debug.WriteLine("The modal page is dismissed");

                UpdateFrame();

                UpdateFeed();

                UpdatePeriods();

            };

            await Navigation.PushModalAsync(modalPage);

        }
```

The below procedure will retrieve the information needed to access the database and assign it to the public SQL Builder variable. Part of the information will be retrieved from the application’s settings, where information can be stored (after the user has logged in) even when the application is not running.

```csharp
        public static void SetDBinfo()

        {

            builder = new SqlConnectionStringBuilder

            {

                DataSource = Settings.CurrentDatabase,

                UserID = "adminDB",

                Password = Settings.CurrentDBPassword,

                InitialCatalog = "aradippou5"

            };

        }
```

The below procedure will get all rows from the database in the Feed table and update the list, the data of which the ListView in the 2nd tab is bound to.

```csharp
        private void UpdateFeed()

        {

            posts = new ObservableCollection<FeedPost>(); //Resetting list of posts

            posts.Clear();

            FeedPost tempPost;

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                String sql = "SELECT Author, DateTimePosted, Post FROM dbo.Feed ORDER BY DateTimePosted DESC;"; //Selecting Only from last week TODO

                using (SqlCommand command = new SqlCommand(sql, connection))

                {

                    connection.Open();

                    using (SqlDataReader reader = command.ExecuteReader())

                    {

                        while (reader.Read())

                        {

                            //Assigning Posts to the List

                            tempPost = new FeedPost(reader.GetString(2), reader.GetString(0), reader.GetDateTime(1));

                            posts.Add(tempPost);

                        }

                        reader.Close();

                    }

                    connection.Close();

                }

                lstFeed.ItemsSource = posts; //binds the source of the listView in the GUI called lstFeed to the list of posts

            }

        }
```

The below procedure will get the logged in user’s “teachings”, AKA their timetable based on the day the application is launched. In the meantime, the appropriate lesson names based on the information in the teachings rows will be retrieved from the Lessons table so that the user can see the string name of lessons in their schedule, not just their numeric identifiers:

```csharp
private void UpdatePeriods()

        {

            CurrentPeriods = new List<int>();

            int day = GetToday();

            List<string> periods = new List<string>();

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                String sql = "SELECT Teachings.PeriodID, Teachings.LessonID, Lessons.LessonName FROM dbo.Teachings " +

                    "INNER JOIN dbo.Lessons ON Teachings.LessonID = Lessons.LessonID WHERE TeacherUsername = '" + Settings.CurrentUsername + "' AND Day = " + day + ";";

                using (SqlCommand command = new SqlCommand(sql, connection))

                {

                    connection.Open();

                    using (SqlDataReader reader = command.ExecuteReader())

                    {

                        while (reader.Read())

                        {

                            periods.Add("Period " + reader.GetByte(0).ToString() + " - " + reader.GetString(2));

                            //CurrentPeriods.Add(reader.GetInt32(0));

                        }

                        reader.Close();

                    }

                    connection.Close();

                }

            }

            pickerGroup.ItemsSource = periods;

        }
```

The below function will return the numeric representation of the week days:

```csharp
        private int GetToday()

        {

            DateTime today = DateTime.Now;

            string day = today.DayOfWeek.ToString();

            switch (day)

            {

                case "Monday":

                    return 1;

                case "Tuesday":

                    return 2;

                case "Wednesday":

                    return 3;

                case "Thursday":

                    return 4;

                case "Friday":

                    return 5;

                case "Saturday":

                    return 6;

                default:

                    return 7;

            }

        }
```

The below procedure will update the text of the Frame on top of the first tab of the Main Page to greet the logged in user:

```csharp
        public void UpdateFrame()

        {

            frameMsg.Text = "Welcome, " + Settings.CurrentUsername + "!";

        }
```

The below procedure will be called once the user clicks on the button btnChangePasswd – it will perform validation before sending the new password to the database.

```csharp
        async void BtnChangePasswd_Clicked(object sender, EventArgs e)

        {

            string currentPassword = GetPasswordCurrent();

            if (currentPassword == entCurrentPasswd.Text)

            {

                if (entNewPasswd.Text == "" || entConfirmPasswd.Text == "")

                {

                    await DisplayAlert("Alert", "New password cannot be empty.", "OK");

                }

                else

                {

                    if (entNewPasswd.Text == entConfirmPasswd.Text)

                    {

                        using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

                        {

                            String sql = "UPDATE Users SET UserPassword='" + entConfirmPasswd.Text + "' WHERE UserName='" + Settings.CurrentUsername + "';";

                            using (SqlCommand command = new SqlCommand(sql, connection))

                            {

                                connection.Open();

                                command.ExecuteNonQuery();

                                connection.Close();

                            }

                        }

                        await DisplayAlert("Success", "Password has been saved.", "OK");

                        entCurrentPasswd.Text = "";

                        entNewPasswd.Text = "";

                        entConfirmPasswd.Text = "";

                    }

                    else

                        await DisplayAlert("Alert", "Passwords Must Match!", "OK");

                }

            }

            else

                await DisplayAlert("Alert", "The current password is incorrect.", "OK");

        }
```

The below procedures will be used to log the user out of the mobile application, set the appropriate setting so that the application requires a user to log in at launch and also push the Log-In screen to the stack of pages so it is visible.

```csharp
        private void BtnLogOut_Clicked(object sender, EventArgs e)

        {

            LogOutAsync();           

        }

        private async void LogOutAsync()

        {

            logoutConfrm = await DisplayAlert("Alert!", "Do you really want to log out?", "Yes", "No");

            if (logoutConfrm == true)

            {

                String sql = "UPDATE dbo.Users SET IsLoggedIn = 0 WHERE UserName = '" + Settings.CurrentUsername + "';";

                using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

                {

                    using (SqlCommand command = new SqlCommand(sql, connection))

                    {

                        connection.Open();

                        command.ExecuteNonQuery();

                        connection.Close();

                    }

                }

                ShowErrorMessage(2);

                Settings.FirstRun = true;

                ShowLogin();

            }

            else

                ShowErrorMessage(3);

        }
```

The below procedure will push the AttendancePage to the stack of pages so it is visible to the user, if the date of the application’s launch exists (checked through the DateExists subroutine). It will also set a variable to the “code-behind” of the AttendancePage so that the user’s period selection is ‘passed’ to that class:

```csharp
        async void BtnTakeAttendance_Clicked(object sender, EventArgs e)

        {

            int periodSelected;

            periodSelected = pickerGroup.SelectedIndex + 1;

            if (DateExists())

            {

                AttendancePage.PeriodPicked = periodSelected;

                AttendancePage attendancePage = new AttendancePage();

                await this.Navigation.PushModalAsync(attendancePage);

            }

            else

                await DisplayAlert("Alert", "There are no lessons for today.", "OK");

        }
```

The below function will check whether the date of the application’s launch exists in the Dates table in the database:

```csharp
        private bool DateExists()

        {

            DateTime today = DateTime.Now;

            int day;

            String sql = "SELECT COUNT(1) FROM dbo.Dates WHERE Date = '" + today.ToString("yyyy-MM-dd") + "';";

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                using (SqlCommand command = new SqlCommand(sql, connection))

                {

                    connection.Open();

                    day = (int)command.ExecuteScalar();

                    connection.Close();

                }

            }

            if (day > 0)

            {

                return true;

            }

            else

                return false;

        }
```

The below function will get the current password of the logged in user from the database, since it is not retained between launches.

```csharp
        private string GetPasswordCurrent()

        {

            string password = "";

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                String sql = "SELECT UserPassword FROM dbo.Users WHERE UserName = '" + Settings.CurrentUsername + "';";

                using (SqlCommand command = new SqlCommand(sql, connection))

                {

                    connection.Open();

                    using (SqlDataReader reader = command.ExecuteReader())

                    {

                        while (reader.Read())

                        {

                            password = reader.GetString(0);

                        }

                        reader.Close();

                    }

                    connection.Close();

                }

            }

            return password;

        }
```

The below procedure will display “pop-ups” when an error occurs. Its numeric parameter determines what type of error will be displayed:

```csharp
        async void ShowErrorMessage(int reason)

        {

            if (reason == 1)

                await DisplayAlert("Alert", "Cannot contact the database. Make sure you are connected to a network.", "OK");

            else if (reason == 2)

                await DisplayAlert("Alert", "You have logged out. Restart of application is recommended.", "OK");

            else if (reason == 3)

                await DisplayAlert("Alert", "You have chosen not to log out.", "OK");

        }       

    }

}
```

After the user selects a period in the first tab and tap on the “Take Attendance” button, the AttendancePage is displayed where the user will take the attendance for that day and period according to their timetable/teachings. The below XAML code designs the ContentPage:

```xml
<?xml version="1.0" encoding="utf-8" ?>

<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"

             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"

             xmlns:input="clr-namespace:Plugin.InputKit.Shared.Controls;assembly=Plugin.InputKit"

             x:Class="csNEA_mobileApp.AttendancePage">

    <ListView x:Name="lstStudents" ItemsSource="{Binding Path=BindingContext.Students}" HorizontalOptions="Center"> <?the binding context is set to the Student class?>

        <ListView.Header>

            <StackLayout Orientation="Horizontal">

                <Button x:Name="btnBack" Text="Go Back" Clicked="BtnBack_Clicked"></Button>

            </StackLayout>

        </ListView.Header>

        <ListView.ItemTemplate>

            <DataTemplate>

                <ViewCell>

                    <ViewCell.View>

                        <StackLayout Orientation="Horizontal"> <?defining objects of the ListView, bound to the Students class?>

                            <Label Text="{Binding FirstName}" FontSize="13" />

                            <Label Text="{Binding LastName}" FontSize="13"/>

                            <input:CheckBox Type="Check" IsChecked="{Binding IsPresent}" HorizontalOptions="EndAndExpand" BorderColor="{AppThemeBinding Dark=White, Light=Black}"/>

                        </StackLayout>

                    </ViewCell.View>

                </ViewCell>

            </DataTemplate>

        </ListView.ItemTemplate>

        <ListView.Footer>

            <StackLayout Orientation="Horizontal" HorizontalOptions="Center">

                <Button x:Name="btnSubmit" Text="Submit Register" BorderWidth="1" Clicked="btnSubmit_Clicked"></Button>

            </StackLayout>

        </ListView.Footer>

    </ListView>

</ContentPage>
```

Once again, data binding is used to link the custom items of the ListView that makes up the page to objects of the Student class. The code that makes up the Students class is as follows:

```csharp
using System;

using System.Collections.Generic;

using System.Text;

namespace csNEA_mobileApp

{

    public class Student

    {

        public Student(int studentID, string firstName, string lastName, bool isPresent)

        {

            StudentID = studentID;

            FirstName = firstName;

            LastName = lastName;

            IsPresent = isPresent;

        }

        public int StudentID { get; set; }

        public string FirstName { get; set; }

        public string LastName { get; set; }

        public bool IsPresent { get; set; }

    }

}
```

The C# ‘code-behind’ of the page follows:

```csharp
using System;

using System.Collections.Generic;

using System.Collections.ObjectModel;

using System.Data.SqlClient;

using System.Linq;

using System.Text;

using System.Threading.Tasks;

using Xamarin.Forms;

using Xamarin.Forms.Xaml;

namespace csNEA_mobileApp

{

    [XamlCompilation(XamlCompilationOptions.Compile)]
```

The following code is run as the class is accessed from the namespace. It is responsible for eventually populating the list teachers must go through to mark students as present or absent. Students will be retrieved either on their own or along with pre-recorded attendance information which may be present if the teacher has filled in the register already.

```csharp
    public partial class AttendancePage : ContentPage

    {

        public static int PeriodPicked { get; set; } //the period as picked from the main page

        public ObservableCollection<Student> Students { get; set; } //similar to a list, will contain all Students for which attendance must be taken

        public static SqlConnectionStringBuilder builder { get; set; } //used to store the database information for this class

        public static bool updateRegister { get; set; } //determines whether attendance has already been taken for that teaching and date

        public string group { get; set; } //the group for which a teaching (part of teachers' timetables) is associated with

 public AttendancePage()

        {

            Students = new ObservableCollection<Student>();

            InitializeComponent();

            SetDBinfo();

            int day = GetToday();

            group = GetTeachingGroup(PeriodPicked, day);

            Students.Clear();

            if (group != "")

            {

                updateRegister = CheckInAbsences(group);

                DateTime today = DateTime.Now;

                if (updateRegister) //the register has been sent already for this teaching and date - get the students along with status submitted

                {

                    Students.Clear();

                    using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

                    {

                        String sql = "SELECT Attendances.StudentID, Attendances.IsPresent, Students.FirstName, Students.LastName " +

                            "FROM Attendances INNER JOIN Students ON Attendances.StudentID = Students.StudentID " +

                            "WHERE StudentGroup='" + group + "' AND Date = '" + today.ToString("yyyy-MM-dd") + "' AND Period = " + PeriodPicked + ";";

                        using (SqlCommand command = new SqlCommand(sql, connection))

                        {

                            connection.Open();

                            using (SqlDataReader reader = command.ExecuteReader())

                            {

                                while (reader.Read())

                                {

                                    Student tempStudent = new Student(reader.GetInt32(0), reader.GetString(2), reader.GetString(3), reader.GetBoolean(1));

                                    Students.Add(tempStudent);

                                }

                                reader.Close();

                            }

                            connection.Close();

                        }

                    }

                }

     else //the register has not been sent for this teaching and date yet - only get the list of students in group associated with teaching data

                {

                    Students.Clear();

                    using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

                    {

                        String sql = "SELECT StudentID, FirstName, LastName FROM dbo.Students WHERE StudentGroup='" + group + "';";

                        using (SqlCommand command = new SqlCommand(sql, connection))

                        {

                            connection.Open();

                            using (SqlDataReader reader = command.ExecuteReader())

                            {

                                while (reader.Read())

                                {

                                    Student tempStudent = new Student(reader.GetInt32(0), reader.GetString(1), reader.GetString(2), false);

                                    Students.Add(tempStudent);

                                }

                                reader.Close();

                            }

                            connection.Close();

                        }

                    }

                } 

      }

            else

            {

                Students.Clear();

                DisplayAlert("Alert", "You do not have a lesson with a group right now.", "OK");               

            }

            this.BindingContext = this;

            lstStudents.ItemsSource = Students;

            //if period correct, check teachings then get students

        }

        private void BtnBack_Clicked(object sender, EventArgs e)

        {

            this.Navigation.PopModalAsync();

        }
```

The following code is run after the teacher has marked students in the list as present or absent. It sends the absence information to the database.

```csharp
       private void btnSubmit_Clicked(object sender, EventArgs e)

        {

            DateTime today = DateTime.Now;

            String sql;

            if (updateRegister)

            {

                for (int i = 0; i < Students.Count; i++)

                {

                    int temp;

                    if (Students[i].IsPresent)

                        temp = 1;

                    else

                        temp = 0;

                    sql = "UPDATE dbo.Attendances SET IsPresent = " + temp + " WHERE Date = '" + today.ToString("yyyy-MM-dd") + "' AND Period = " + PeriodPicked + " AND StudentID = " + Students[i].StudentID + " AND MemberOf = '" + group + "';";

                    using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

                    {

                        using (SqlCommand command = new SqlCommand(sql, connection))

                        {

                            connection.Open();

                            command.ExecuteNonQuery();

                            connection.Close();

                        }

                    }

                }

            }

            else

            {

                for (int i = 0; i < Students.Count; i++)

                {

                    int temp;

                    if (Students[i].IsPresent)

                        temp = 1;

                    else

                        temp = 0;

                    sql = "INSERT INTO dbo.Attendances (Date, Period, StudentID, MemberOf, IsPresent) " +

                        "Values ('" + today.ToString("yyyy-MM-dd") + "', " + PeriodPicked + ", " + Students[i].StudentID + ", '"+ group +"', " + temp +");";

                    using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

                    {

                        using (SqlCommand command = new SqlCommand(sql, connection))

                        {

                            connection.Open();

                            command.ExecuteNonQuery();

                            connection.Close();

                        }

                    }

                }

            }

            this.Navigation.PopModalAsync();
        }
```

The following procedure keeps the database information temporarily to be accessed by the class:

```csharp
 public static void SetDBinfo()

        {

            builder = new SqlConnectionStringBuilder

            {

                DataSource = Settings.CurrentDatabase,

                UserID = "adminDB",

                Password = Settings.CurrentDBPassword,

                InitialCatalog = "aradippou5"

            };

        }
```

The following function returns the numeric identification of today’s day based on DateTime:

```csharp
        private int GetToday()

        {

            DateTime today = DateTime.Now;

            string day = today.DayOfWeek.ToString();

            switch (day)

            {

                case "Monday":

                    return 1;

                case "Tuesday":

                    return 2;

                case "Wednesday":

                    return 3;

                case "Thursday":

                    return 4;

                case "Friday":

                    return 5;

                case "Saturday":

                    return 6;

                default:

                    return 7;

            }

        }
```

The following function returns the group that the teacher is teaching for the passed period and day numeric representations:

```csharp
private string GetTeachingGroup(int period, int day)

        {

            string output = "";

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                String sql = "SELECT [Group] FROM dbo.Teachings WHERE PeriodID = " + period + " AND Day = " + day + " AND TeacherUsername = '" + Settings.CurrentUsername + "';";

                using (SqlCommand command = new SqlCommand(sql, connection))

                {

                    connection.Open();

                    using (SqlDataReader reader = command.ExecuteReader())

                    {

                        while (reader.Read())

                        {

                            if (reader.IsDBNull(0))

                                output = "";

                            else

                                output = reader.GetString(0);

                        }

                        reader.Close();

                    }

                    connection.Close();

                }

            }

            return output;

        }
```

The following function checks whether absence information for the group returned from the function above has been recorded already in the database:

```csharp
private bool CheckInAbsences(string group) //determines whether the list of students will be later retrieved from the absences table or the students table

        {

            DateTime today = DateTime.Now;

            int records;

            String sql = "SELECT COUNT(1) FROM dbo.Attendances WHERE MemberOf = '" + group + "' AND Date = '" + today.ToString("yyyy-MM-dd") + "' AND Period = " + PeriodPicked + ";";

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                using (SqlCommand command = new SqlCommand(sql, connection))

                {

                    connection.Open();

                    records = (int)command.ExecuteScalar();

                    connection.Close();

                }

            }

            if (records > 0) //means attendances have already been taken for the current teaching and date

            {

                return true;

            }

            else

                return false;

        }

        protected override bool OnBackButtonPressed()

        {

            return true;

        }

    }
```
