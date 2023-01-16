# Implementing the Desktop Application

This marks the start of the “Register” form used by administrators to add new users:

```csharp
using System;

using System.Collections.Generic;

using System.ComponentModel;

using System.Data;

using System.Drawing;

using System.Linq;

using System.Text;

using System.Threading.Tasks;

using System.Windows.Forms;

using System.IO;

using System.Data.SqlClient;

using System.Security.Cryptography;

namespace csNEA

{

    public partial class Register : Form

    {

        public static SqlConnectionStringBuilder builder { get; set; }

        public static UnicodeEncoding ByteConverter = new UnicodeEncoding();

        public static RSACryptoServiceProvider RSA = new RSACryptoServiceProvider();

        public static byte[] plaintext;

        public static byte[] encryptedtext;

        public Register()

        {

            InitializeComponent();

        }
```

The below procedure hides the registration form and returns the user to the administrator form:

```csharp
        private void btnBack_Click(object sender, EventArgs e)

        {

            this.Visible = false;

            frmAdmin admin = new frmAdmin();

            admin.UpdateUsersList();

            admin.ShowDialog();

        }
```

The below procedure sets the database information to the form’s temporary SQL builder variable:

```csharp
        public static void SetDBinfo(SqlConnectionStringBuilder input)
        {
            builder = input;
        }
```

The below procedure adds the user to the database:

```csharp
        private void btnAddUser_Click(object sender, EventArgs e)

        {

            string rights;

            if (txtNewUsername == null || txtNewPassword.Text == null || txtFirstName == null)

                MessageBox.Show("The first three fields must not be empty!");

            else

            {

                if (cmbRights.Text == "Administrator")

                {

                    rights = "a";

                }

                else if (cmbRights.Text == "Secretary")

                {

                    rights = "s";

                }

                else

                {

                    rights = "t";

                }

                string cmd;

                plaintext = ByteConverter.GetBytes(txtNewPassword.Text);

                encryptedtext = PasswordEncryption.Encryption(plaintext, RSA.ExportParameters(false), false);

                using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

                {

                    if (txtLastName.Text != null)

                        cmd = "Insert into Users (UserName, UserPassword, UserRole, FirstName, LastName, IsLoggedIn) " +

                            "Values('" + txtNewUsername.Text + "', @Content, '" + rights + "', '" + txtFirstName.Text + "', '" + txtLastName.Text + "', '0');";

                    else

                        cmd = "Insert into Users (UserName, UserPassword, UserRole, FirstName, IsLoggedIn) " +

                            "Values('" + txtNewUsername.Text + "', @Content, '" + rights + "', '" + txtFirstName.Text + "', '0');";

                    String sql = cmd;

                    using (SqlCommand command = new SqlCommand(sql, connection))

                    {

                        SqlParameter param = command.Parameters.Add("@Content", SqlDbType.VarBinary);

                        param.Value = encryptedtext;

                        connection.Open();

                        command.ExecuteNonQuery();

                        connection.Close();

                    }

                }

                MessageBox.Show("User added successfuly.");

            }

        }

    }
}
```

This marks the start of the “RegisterStudent” form used by administrators to add new students:

```csharp
using System;

using System.Collections.Generic;

using System.ComponentModel;

using System.Data;

using System.Data.SqlClient;

using System.Drawing;

using System.Linq;

using System.Text;

using System.Threading.Tasks;

using System.Windows.Forms;

namespace csNEA

{

    public partial class RegisterStudent : Form

    {

        public static SqlConnectionStringBuilder builder { get; set; }

        public RegisterStudent()

        {

            InitializeComponent();

        }
```

The below procedure sets the database information to the form’s temporary SQL builder variable:

```csharp
        public static void SetDBinfo(SqlConnectionStringBuilder input)

        {
            builder = input;
        }
```

The below procedure adds the student to the database:

```csharp
        private void btnAddStudent_Click(object sender, EventArgs e)

        {

            int dump;

            if (txtFirstName.Text == null || txtLastName.Text == null || cmbGroups.Text == null || txtMotherName.Text == null || txtFatherName.Text == null || !int.TryParse(txtMotherPhone.Text, out dump) || !int.TryParse(txtFatherPhone.Text, out dump) || !int.TryParse(txtThirdNo.Text, out dump) || txtThirdName.Text == null)

            {

                MessageBox.Show("Make sure all fields are filled in, and that valid phone numbers are provided without spaces and try again.");

            }

            else

            {                

                try

                {

                    using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

                    {

                        String sql;

                        if (txtThirdRole.Text == null)

                        {

                            sql = "Insert into Students (FirstName, LastName, StudentGroup, MotherName, MotherPhone, FatherName, FatherPhone, ThirdName, ThirdRole, ThirdPhone) " +

                            "Values('" + txtFirstName.Text + "', '" + txtLastName.Text + "', '" + cmbGroups.Text + "', '" + txtMotherName.Text + "', " + txtMotherPhone.Text + ", '" + txtFatherName.Text + "', " + txtFatherPhone.Text + ", '" + txtThirdName.Text + "', '" + txtThirdRole.Text + "', " + txtThirdNo.Text + ");";

                        }

                        else

                        {

                            sql = "Insert into Students (FirstName, LastName, StudentGroup, MotherName, MotherPhone, FatherName, FatherPhone, ThirdName, ThirdPhone) " +

                            "Values('" + txtFirstName.Text + "', '" + txtLastName.Text + "', '" + cmbGroups.Text + "', '" + txtMotherName.Text + "', " + txtMotherPhone.Text + ", '" + txtFatherName.Text + "', " + txtFatherPhone.Text + ", '" + txtThirdName.Text + "', " + txtThirdNo.Text + ");";

                        }

                        using (SqlCommand command = new SqlCommand(sql, connection))

                        {

                            connection.Open();

                            command.ExecuteNonQuery();

                            connection.Close();

                        }

                    }

                    MessageBox.Show("Student added successfuly.");

                }

                catch

                {

                    MessageBox.Show("Information Provided is Incorrect.");

                }

            }            

        }
```

The below procedure is run when the form is displayed. It gets the groups from the database and fills a ComboBox so that the user can select which group the student will be a member of:

```csharp
        private void RegisterStudent_Load(object sender, EventArgs e)

        {

            cmbGroups.Items.Clear();            

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                String sql = "SELECT GroupID FROM dbo.Groups";

                using (SqlCommand command = new SqlCommand(sql, connection))

                {

                    connection.Open();

                    using (SqlDataReader reader = command.ExecuteReader())

                    {

                        while (reader.Read())

                        {

                            cmbGroups.Items.Add(reader.GetString(0));                          

                        }

                        reader.Close();

                    }

                    connection.Close();

                }

            }

        }
```

The below procedure hides the student registration form and returns the user to the administrator form:

```csharp
        private void btnBack_Click(object sender, EventArgs e)

        {

            this.Visible = false;

            frmAdmin admin = new frmAdmin();

            //update students list not necessary

            admin.Show();

        }

    }

}
```

This marks the start of the “Login” form used to log into the desktop application:

```csharp
using System;

using System.Collections.Generic;

using System.ComponentModel;

using System.Data;

using System.Drawing;

using System.Linq;

using System.Text;

using System.Threading.Tasks;

using System.Windows.Forms;

using System.IO;

using System.Data.SqlClient;

using csNEA.Properties;

using System.Security.Cryptography;

namespace csNEA

{   

    public partial class frmLogin : Form

    {        

        List<User> listOfUsers = new List<User>() { };       

        User tempUser = new User();

        public static string passUser;

        public static char passRights;

        public frmLogin()

        {                                  

            InitializeComponent();

        }
```

The below procedure is run when the form is displayed. It fetches the information stored in the application’s settings if, on the previous run the user has chosen to store the database information:

```csharp
        private void frmLogin_Load(object sender, EventArgs e)

        {

            if (Settings.Default.DBaddress != string.Empty)

            {

                txtDatabase.Text = Settings.Default.DBaddress;

                txtDBPassword.Text = Settings.Default.DBpassword;

            }

        }
```

The below procedure fetches the users from the database and checks them with the credentials entered by the user. It then responds accordingly, based on the role (access rights) of the user detected:

```csharp
        private void btnConnect_Click(object sender, EventArgs e)

        {

            bool success = false;

            SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder();

            builder.DataSource = txtDatabase.Text;

            builder.UserID = "adminDB";

            builder.Password = txtDBPassword.Text;

            builder.InitialCatalog = "aradippou5";

            if (txtUser.Text == "" || txtPassword.Text == "" || txtDatabase.Text == "" || txtDBPassword.Text == "")

                MessageBox.Show("No fields can be empty!");

            else

            {

                using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

                {

                    String sql = "SELECT UserName, UserPassword, UserRole FROM dbo.Users";

                    using (SqlCommand command = new SqlCommand(sql, connection))

                    {

                        connection.Open();

                        using (SqlDataReader reader = command.ExecuteReader())

                        {

                            while (reader.Read())

                            {

                                //each user is fetched here

                                AddUser(reader.GetString(0), reader.GetString(1), reader.GetString(2));

                            }

                            reader.Close();

                        }

                        connection.Close();

                    }

                }

                for (int i = 0; i < listOfUsers.Count; i++)

                {

                    if (txtUser.Text == listOfUsers[i].username && txtPassword.Text == listOfUsers[i].password)

                    {

                        success = true;

                        passUser = listOfUsers[i].username;

                        if (chckRemember.Checked)

                        {

                            Settings.Default.DBaddress = txtDatabase.Text;

                            Settings.Default.DBpassword = txtDBPassword.Text;

                            Settings.Default.Save();

                        }

                        else

                        {

                            Settings.Default.DBaddress = "";

                            Settings.Default.DBpassword = "";

                            Settings.Default.Save();

                        }

                        this.Visible = false;

                        if (listOfUsers[i].accessRights == "s")

                        {

                            passRights = 's';

                            listOfUsers.Clear();

                            this.Visible = false;

                            frmAdmin admin = new frmAdmin();

                            frmAdmin.SetDBinfo(txtDatabase.Text, txtDBPassword.Text);

                            admin.ShowDialog();

                        }

                        else if (listOfUsers[i].accessRights == "a")

                        {

                            frmAdmin.currentUser = listOfUsers[i].username;

                            passRights = 'a';

                            listOfUsers.Clear();

                            this.Visible = false;

                            frmAdmin admin = new frmAdmin();

                            frmAdmin.SetDBinfo(txtDatabase.Text, txtDBPassword.Text);

                            admin.ShowDialog();

                        }

                        else

                        {

                            listOfUsers.Clear();

                            MessageBox.Show("Teachers do not have access to the desktop application!");

                        }

                    }

                }

                if (success == false)

                {

                    MessageBox.Show("Login Failed");

                    listOfUsers.Clear();

                }

            }                           

        }
```

The below procedure adds a user to the list of users. Its parameters are passed from what the reader above fetches from the database:

```csharp
        private void AddUser(string username, string password, string rights)

        {

            tempUser = new User();

            tempUser.username = username;

            tempUser.password = password;

            tempUser.accessRights = rights;

            listOfUsers.Add(tempUser);

        }

        protected override void OnFormClosing(FormClosingEventArgs e)

        {

            e.Cancel = false;

            base.OnFormClosing(e);

            Application.Exit();

        }

    }

    class User

    {

        public string username;

        public string password;

        public string accessRights;

    }

}
```

What follows is the administrator, or main form implemented in C#:

```csharp
using System;

using System.Collections.Generic;

using System.ComponentModel;

using System.Data;

using System.Drawing;

using System.Linq;

using System.Text;

using System.Threading.Tasks;

using System.Windows.Forms;

using Microsoft.VisualBasic;

using System.IO;

using System.Data.SqlClient;

using CsvHelper;

using System.Globalization;

namespace csNEA

{

    public partial class frmAdmin : Form

    {

        public static SqlConnectionStringBuilder builder { get; set; }

        public static string currentUser { get; set; }

        public frmAdmin()

        {

            InitializeComponent();

        }
```

The below procedure sets the database information to the form’s temporary SQL builder variable:

```csharp
        public static void SetDBinfo(string input, string password)

        {

            builder = new SqlConnectionStringBuilder

            {

                DataSource = input,

                UserID = "adminDB",

                Password = password,

                InitialCatalog = "aradippou5"

            };

        }

        private void btnRegister_Click(object sender, EventArgs e)

        {

            this.Visible = false;

            Register register = new Register();

            Register.SetDBinfo(builder);

            register.ShowDialog();

        }
```

The below procedure returns the user to the LogIn Form:

```csharp
        private void btnLogOut_Click(object sender, EventArgs e)

        {

            this.Visible = false;            

            frmLogin logIn = new frmLogin();

            logIn.ShowDialog();

        }
```

The below procedure runs when the form is displayed and updates the initial ListBoxes:

```csharp
        private void frmAdmin_Load(object sender, EventArgs e)

        {

            //reload all lists

            if (frmLogin.passRights == 'a')

            {

                UpdateUsersList();

                UpdateLessonsList();

                UpdateSemList();

            }

            UpdateGroupsList();

        }
```

The below procedure does not allow secretaries to view specific tabs in the desktop application:

```csharp
        private void adminTabs_SelectedIndexChanged(object sender, System.EventArgs e)

        {

            if (frmLogin.passRights == 's' && (adminTabs.SelectedTab == usersPage || adminTabs.SelectedTab == tabLessons || adminTabs.SelectedTab == datesPage))

            {

                MessageBox.Show("Unable to load tab. You have insufficient access privileges.");

                adminTabs.SelectedTab = tabAbsences;

            }

        }

        protected override void OnFormClosing(FormClosingEventArgs e)

        {

            e.Cancel = false;

            base.OnFormClosing(e);

            Application.Exit();

        }
```

The below procedure posts an update to the feed:

```csharp
        private void btnFeedPost_Click(object sender, EventArgs e)

        {

            DateTime rightNow = DateTime.Now;

            string input = Interaction.InputBox("Type your feed post below:", "Posting to the Feed", "", 0, 0);

            if (input == "")

            {

                MessageBox.Show("The post will not be uploaded."); //include test in word document

            }

            else

            {

                String sql = "Insert into Feed (Author, DateTimePosted, Post) " +

                "Values('" + currentUser + "', '" + rightNow.ToString("yyyy-MM-dd HH:mm:ss") + "', '" + input + "');";

                ExecuteSQLInsert(sql);

                MessageBox.Show("Post Uploaded Successfully.");

            }

        }
```

The below procedure uploads a user’s schedule:

```csharp
        private void btnSchedule_Click(object sender, EventArgs e)

        {

            //the user will first select the file with lessons then corresponding file with groups the lessons are taught to

            if (lstUsers.SelectedItems.Count == 1)

            {

                MessageBox.Show("WARNING: This will override the current schedule of the user (if any exists).");

                uploadSchDiag.ShowDialog();

                int[,] schedule = new int[7, 5];

                string[,] schGroups = new string[7, 5];

                string scheduleFile = uploadSchDiag.FileName;

                try

                {

                    StreamReader reader = new StreamReader(scheduleFile);

                    int i = 0;

                    while (!reader.EndOfStream && i < 7)

                    {

                        var line = reader.ReadLine();

                        var values = line.Split(',');

                        for (int j = 0; j < 5; j++)

                        {

                            schedule[i, j] = int.Parse(values[j]);

                        }

                        i++;

                    }

                    reader.Close();

                    MessageBox.Show("Now select the corresponding Groups file.");

                    uploadSchDiag.ShowDialog();

                    string groupsFile = uploadSchDiag.FileName;

                    reader = new StreamReader(groupsFile);

                    i = 0;

                    while (!reader.EndOfStream && i < 7)

                    {

                        var line = reader.ReadLine();

                        var values = line.Split(',');

                        for (int j = 0; j < 5; j++)

                        {

                            schGroups[i, j] = values[j];

                        }

                        i++;

                    }

                    reader.Close();

                    UploadSchedule(schedule, schGroups);

                }

                catch

                {

                    MessageBox.Show("You have not selected an appropriate file.");

                }

            }

            else

                MessageBox.Show("You must first select a user from the list!");

        }

        private void UploadSchedule(int[,] sc, string[,] scGrp)

        {

            int rowsAffected = 0;

            try

            {

                string selectedUser = lstUsers.SelectedItems[0].Text;

                ClearSchedule(selectedUser);

                int m, k; //increments day and period values to match database

                for (int i = 0; i < 5; i++)

                {

                    k = i + 1;

                    for (int j = 0; j < 7; j++)

                    {

                        m = j + 1;

                        using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

                        {

                            String sql;

                            if (scGrp[j, i] == "NULL")

                            {

                                sql = "Insert into Teachings (PeriodID, LessonID, TeacherUsername, Day, [Group]) Values(" + m + ", " + sc[j, i] + ", '" + selectedUser + "', " + k + ", NULL);";

                            }

                            else

                            {

                                sql = "Insert into Teachings (PeriodID, LessonID, TeacherUsername, Day, [Group]) Values(" + m + ", " + sc[j, i] + ", '" + selectedUser + "', " + k + ", '" + scGrp[j, i] + "');";

                            }

                            using (SqlCommand command = new SqlCommand(sql, connection))

                            {

                                connection.Open();

                                rowsAffected = rowsAffected + command.ExecuteNonQuery();

                                connection.Close();

                            }

                        }

                    }

                }

                MessageBox.Show(rowsAffected + " teachings were uploaded successfully.");

            }

            catch

            {

                if (rowsAffected == 0)

                    MessageBox.Show("You have not selected a user from the list!");

                else

                    MessageBox.Show("The information provided in the file does not match the registered Lessons. Teachings created for user = " + rowsAffected + ".");

            }

        }
```

The below procedure removes any teachings information from the database from a user before the new one is uploaded:

```csharp
        private void ClearSchedule(string user)

        {

            String sql = "Delete from Teachings WHERE TeacherUsername='" + user + "';";

            ExecuteSQLInsert(sql);

        }
```

The below procedure removes all users from the listBox and fetches them again from the database:

```csharp
        public void UpdateUsersList()

        {

            lstUsers.Items.Clear();

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                String sql = "SELECT UserName, FirstName, LastName, UserRole, IsLoggedIn FROM dbo.Users";

                using (SqlCommand command = new SqlCommand(sql, connection))

                {

                    connection.Open();

                    using (SqlDataReader reader = command.ExecuteReader())

                    {

                        while (reader.Read())

                        {

                            //Console.WriteLine("{0} {1}", reader.GetString(0), reader.GetString(1));

                            ListViewItem user = new ListViewItem(reader.GetString(0));

                            user.SubItems.Add(reader.GetString(1));

                            if (!reader.IsDBNull(2))

                            {

                                user.SubItems.Add(reader.GetString(2));

                            }

                            user.SubItems.Add(reader.GetString(3));

                            user.SubItems.Add(reader.GetBoolean(4).ToString());

                            lstUsers.Items.Add(user);

                        }

                        reader.Close();

                    }

                    connection.Close();

                }

            }

        }
```

The below procedure removes all lessons from the listBox and fetches them again from the database:

```csharp
        private void UpdateLessonsList()

        {

            lstLessons.Items.Clear();

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                String sql = "SELECT LessonID, LessonName FROM dbo.Lessons";

                using (SqlCommand command = new SqlCommand(sql, connection))

                {

                    connection.Open();

                    using (SqlDataReader reader = command.ExecuteReader())

                    {

                        while (reader.Read())

                        {

                            //Console.WriteLine("{0} {1}", reader.GetString(0), reader.GetString(1));

                            int x = (byte)reader[0];

                            ListViewItem lesson = new ListViewItem(x.ToString());

                            lesson.SubItems.Add(reader.GetString(1));

                            lstLessons.Items.Add(lesson);

                        }

                        reader.Close();

                    }

                    connection.Close();

                }

            }

        }
```

The below procedure validates and adds a new lesson to the database:

```csharp
        private void btnAddLesson_Click(object sender, EventArgs e)

        {

            int id;

            if (!int.TryParse(txtLessonID.Text, out id) || txtLesson.Text == null)

            {

                MessageBox.Show("The Lesson ID must be an Integer and the Name must not be Null.");

            }

            else

            {

                String sql = "Insert into Lessons (LessonID, LessonName) " +

                    "Values(" + txtLessonID.Text + ", '" + txtLesson.Text + "');";

                ExecuteSQLInsert(sql);

                UpdateLessonsList();

            }

        }
```

The below procedure removes the selected lesson and all records associated with it such as teaching records from the database:

```csharp
        private void btnDelLesson_Click(object sender, EventArgs e)

        {

            if (lstLessons.Items.Count > 0 && lstLessons.SelectedItems.Count == 1)

            {

                string lessonToDelete = lstLessons.SelectedItems[0].Text;

                String sql = "DELETE FROM Lessons WHERE LessonID=" + lessonToDelete + ";";

                ExecuteSQLInsert(sql);

                sql = "DELETE FROM Teachings WHERE LessonID =" + lessonToDelete + ";";

                ExecuteSQLInsert(sql);

                MessageBox.Show("Lesson removed successfuly.");

                UpdateLessonsList();

            }

            else

                MessageBox.Show("You need to have a lesson from the list selected first.");

        }
```

The below procedure is used to change a user’s password:

```csharp
        private void btnChangePassword_Click(object sender, EventArgs e)

        {           

            if (lstUsers.SelectedItems.Count == 1) //starting with validation of selected items in the listbox

            {

                string username = lstUsers.SelectedItems[0].Text;

                string input = Interaction.InputBox("Type the new password below:", "Changing Password for " + username, "", 0, 0);

                string confirm = Interaction.InputBox("Repeat the new password below:", "Changing Password for " + username, "", 0, 0);

                if (input == "" || confirm == "")

                {

                    MessageBox.Show("New password cannot be empty!");

                }

                else

                {

                    if (input == confirm)

                    {

                        String sql = "UPDATE Users SET UserPassword='" + input + "' WHERE UserName='" + username + "';";

                        ExecuteSQLInsert(sql);

                        MessageBox.Show("Password for " + username + " changed successfuly.");

                    }

                    else

                        MessageBox.Show("Passwords Do Not Match!");

                }

            }

            else

                MessageBox.Show("You must first select a user from the list!");

        }
```

The below procedure validates and adds a group to the database:

```csharp
        private void btnAddGroup_Click(object sender, EventArgs e)

        {

            if (txtGroup.Text == null)

            {

                MessageBox.Show("The Group ID must not be Null.");

            }

            else

            {

                String sql = "Insert into Groups (GroupID) Values('" + txtGroup.Text + "');";

                ExecuteSQLInsert(sql);

                UpdateGroupsList();

            }

        }
```

The below procedure removes all groups from the relevant listBoxes and comboBoxes and fetches them again from the database:

```csharp
        public void UpdateGroupsList()

        {

            lstGroups.Items.Clear();

            cmbInitialGroup.Items.Clear();

            cmbFinalGroup.Items.Clear();

            cmbGroups.Items.Clear();

            cmbAbsentGroups.Items.Clear();

            cmbMentors.Items.Clear();

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                String sql = "SELECT GroupID, TeacherRep FROM dbo.Groups";

                using (SqlCommand command = new SqlCommand(sql, connection))

                {

                    connection.Open();

                    using (SqlDataReader reader = command.ExecuteReader())

                    {

                        while (reader.Read())

                        {

                            ListViewItem group = new ListViewItem(reader.GetString(0));

                            cmbGroups.Items.Add(reader.GetString(0));

                            cmbInitialGroup.Items.Add(reader.GetString(0));

                            cmbMentors.Items.Add(reader.GetString(0));

                            cmbAbsentGroups.Items.Add(reader.GetString(0));

                            cmbFinalGroup.Items.Add(reader.GetString(0));

                            if (!reader.IsDBNull(1))

                            {

                                group.SubItems.Add(reader.GetString(1));

                            }

                            lstGroups.Items.Add(group);

                        }

                        reader.Close();

                    }

                    connection.Close();

                }

            }

        }
```

The below procedure sets a user as the ‘head’, or mentor of a group:

```csharp
        private void btnMentorSet_Click(object sender, EventArgs e)

        {

            if (lstUsers.SelectedItems.Count == 1)

            {

                string group = cmbMentors.Text;

                string userSelected = lstUsers.SelectedItems[0].Text;

                if (group != null)

                {

                    String sql = "UPDATE Groups SET TeacherRep = null WHERE TeacherRep = '" + userSelected + "';"; //Removes user as Representative of any other group.

                    ExecuteSQLInsert(sql);

                    sql = "UPDATE Groups SET TeacherRep = '" + userSelected + "' WHERE GroupID='" + group + "';"; //Assigns user as Representative of a group.

                    ExecuteSQLInsert(sql);

                    MessageBox.Show("Group properties saved successfuly.");

                    cmbMentors.Text = "";

                    UpdateGroupsList(); //maybe

                }

                else

                    MessageBox.Show("You have not selected a group. If there is none, you can add groups in the Groups Management tab.");

            }

            else

                MessageBox.Show("You have not selected a user from the list!");

        }
```

The below procedure adds all students found in a flat file:

```csharp
        private void btnUploadStudents_Click(object sender, EventArgs e)

        {

            List<Student> students = new List<Student>();

            Student tempStudent;

            uploadStudentsDiag.ShowDialog();

            try

            {

                string studentsFile = uploadStudentsDiag.FileName;

                StreamReader reader = new StreamReader(studentsFile);

                students.Clear();

                while (!reader.EndOfStream)

                {

                    var line = reader.ReadLine();

                    var values = line.Split(',');

                    tempStudent = new Student(values[0], values[1], values[2], values[3], values[4], values[5], values[6], int.Parse(values[7]), int.Parse(values[8]), int.Parse(values[9]));

                    students.Add(tempStudent);

                }

                UploadStudents(students);

                MessageBox.Show("Students Uploaded Successfuly.");

            }

            catch

            {

                MessageBox.Show("You have not selected an appropriate file.");

            }

        }

        private void UploadStudents(List<Student> students)

        {

            for (int i = 0; i < students.Count; i++)

            {

                String sql = "Insert into Students (FirstName, LastName, StudentGroup, MotherName, MotherPhone, FatherName, FatherPhone, ThirdName, ThirdRole, ThirdPhone) " +

                    "Values('" + students[i].FirstName + "', '" + students[i].LastName + "', '" + students[i].Group + "', '" + students[i].MothersName + "', " + students[i].MotherPhone.ToString() + ", '" + students[i].FathersName + "', " + students[i].FatherPhone.ToString() + ", '" + students[i].ThirdName + "', '" + students[i].ThirdRole + "', " + students[i].ThirdPhone.ToString() + ");";

                ExecuteSQLInsert(sql);

            }

        }
```

The below procedure displays the RegisterStudent form where students can be added manually:

```csharp
        private void btnStudentManualAdd_Click(object sender, EventArgs e)

        {

            this.Visible = false;

            RegisterStudent registerStudent = new RegisterStudent();

            RegisterStudent.SetDBinfo(builder);

            registerStudent.ShowDialog();

        }
```

The below procedure displays all students in a given group:

```csharp
        private void btnDisGroup_Click(object sender, EventArgs e)

        {

            if (cmbGroups.Text == "")

                MessageBox.Show("You must first select a group!");

            else

            {

                lstStudents.Items.Clear();

                using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

                {

                    String sql = "SELECT StudentID, FirstName, LastName, StudentGroup, MotherName, MotherPhone, FatherName, FatherPhone, ThirdName, ThirdRole, ThirdPhone FROM dbo.Students WHERE StudentGroup='" + cmbGroups.Text + "';";

                    using (SqlCommand command = new SqlCommand(sql, connection))

                    {

                        connection.Open();

                        using (SqlDataReader reader = command.ExecuteReader())

                        {

                            while (reader.Read())

                            {

                                ListViewItem student = new ListViewItem(reader.GetInt32(0).ToString()); //ID

                                student.SubItems.Add(reader.GetString(1)); //FirstName

                                student.SubItems.Add(reader.GetString(2)); //LastName

                                student.SubItems.Add(reader.GetString(3)); //Group

                                student.SubItems.Add(reader.GetString(4)); //Mother's Name

                                student.SubItems.Add(reader.GetInt32(5).ToString()); //Mother's Phone Number                            

                                student.SubItems.Add(reader.GetString(6)); //Father's Name

                                student.SubItems.Add(reader.GetInt32(7).ToString()); //Father's Phone Number

                                student.SubItems.Add(reader.GetString(8)); //Backup Contact's Name

                                if (!reader.IsDBNull(9))

                                    student.SubItems.Add(reader.GetString(9)); //Backup Contact's Role

                                student.SubItems.Add(reader.GetInt32(10).ToString()); //Backup Contact's Phone Number                            

                                lstStudents.Items.Add(student);

                            }

                            reader.Close();

                        }

                        connection.Close();

                    }

                }

            }            

        }
```

The below procedure displays all students:

```csharp
        private void btnShowAllStudents_Click(object sender, EventArgs e)

        {

            lstStudents.Items.Clear();

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                String sql = "SELECT StudentID, FirstName, LastName, StudentGroup, MotherName, MotherPhone, FatherName, FatherPhone, ThirdName, ThirdRole, ThirdPhone FROM dbo.Students;";

                using (SqlCommand command = new SqlCommand(sql, connection))

                {

                    connection.Open();

                    using (SqlDataReader reader = command.ExecuteReader())

                    {

                        while (reader.Read())

                        {

                            ListViewItem student = new ListViewItem(reader.GetInt32(0).ToString()); //ID

                            student.SubItems.Add(reader.GetString(1)); //FirstName

                            student.SubItems.Add(reader.GetString(2)); //LastName

                            student.SubItems.Add(reader.GetString(3)); //Group

                            student.SubItems.Add(reader.GetString(4)); //Mother's Name

                            student.SubItems.Add(reader.GetInt32(5).ToString()); //Mother's Phone Number

                            student.SubItems.Add(reader.GetString(6)); //Father's Name

                            student.SubItems.Add(reader.GetInt32(7).ToString()); //Father's Phone Number

                            student.SubItems.Add(reader.GetString(8)); //Backup Contact's Name

                            if (!reader.IsDBNull(9))

                                student.SubItems.Add(reader.GetString(9)); //Backup Contact's Role

                            student.SubItems.Add(reader.GetInt32(10).ToString()); //Backup Contact's Phone Number

                            //if (!reader.IsDBNull(1))

                            lstStudents.Items.Add(student);

                        }

                        reader.Close();

                    }

                    connection.Close();

                }

            }

        }
```

The below procedure updates the group of students (transfers them) to another group:

```csharp
The below procedure updates the group of students (transfers them) to another group:

        private void btnTransferStudents_Click(object sender, EventArgs e)

        {

            if (cmbFinalGroup.Text == null || cmbInitialGroup.Text == null)

                MessageBox.Show("You must select both groups!");

            else

            {

                String sql = "UPDATE Students SET StudentGroup = '" + cmbFinalGroup.Text + "' WHERE StudentGroup='" + cmbInitialGroup.Text + "';";

                ExecuteSQLInsert(sql);

                lstStudents.Items.Clear();

                MessageBox.Show("Done!");

                cmbInitialGroup.Text = "";

                cmbFinalGroup.Text = "";

            }

        }
```

The below procedure adds a semester and dates to the database:

```csharp
        private void btnAddSemester_Click(object sender, EventArgs e)

        {

            bool semPresent = false;

            int semNo;

            if (!int.TryParse(txtSemNo.Text, out semNo))

            {

                MessageBox.Show("The semester ID must be a number.");

            }

            else

            {

                if (DateTime.Compare(dtInitial.Value, dtFinal.Value) <= 0)

                {

                    DateTime start = dtInitial.Value.Date;

                    DateTime end = dtFinal.Value.Date;

                    for (int i = 0; i < lstSemesters.Items.Count; i++) //validation for semester ID input

                    {

                        if (lstSemesters.Items[i].Text == txtSemNo.Text)

                        {

                            semPresent = true;

                            break;

                        }

                    }

                    if (!semPresent)

                    {

                        SetSemester(start.ToString("yyyy-MM-dd"), end.ToString("yyyy-MM-dd"), semNo);

                        for (var dt = start; dt <= end; dt = dt.AddDays(1))

                        {

                            UploadDate(dt, semNo);

                        }

                        MessageBox.Show("Semester Created Successfully and Dates were created.");

                        UpdateSemList();

                    }

                    else

                        MessageBox.Show("This semester ID has already been used, enter a new semester ID.");

                }

                else

                {

                    MessageBox.Show("The initial date cannot be later than the final date!");

                }

            }

        }

        private void SetSemester(string start, string end, int semester)

        {

            String sql = "Insert into Semesters (SemesterNumber, StartDate, EndDate) Values(" + semester + ", '" + start + "', '" + end + "');";

            ExecuteSQLInsert(sql);

        }

        private void UploadDate(DateTime date, int semester)

        {

            string dateString = date.ToString("yyyy-MM-dd");

            string day = date.DayOfWeek.ToString();

            int dayToInt;

            switch (day)

            {

                case "Monday":

                    dayToInt = 1;

                    break;

                case "Tuesday":

                    dayToInt = 2;

                    break;

                case "Wednesday":

                    dayToInt = 3;

                    break;

                case "Thursday":

                    dayToInt = 4;

                    break;

                case "Friday":

                    dayToInt = 5;

                    break;

                case "Saturday":

                    dayToInt = 6;

                    break;

                default:

                    dayToInt = 7;

                    break;

            }

            String sql = "Insert into Dates (Date, SemesterNumber, DayNumber, IsHoliday) Values('" + dateString + "', " + semester + ", " + dayToInt + ", 0);";

            ExecuteSQLInsert(sql);

        }
```

The below procedure removes all semesters from the listBox and fetches them again from the database:

```csharp
        private void UpdateSemList()

        {

            lstSemesters.Items.Clear();

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                String sql = "SELECT SemesterNumber, StartDate, EndDate FROM dbo.Semesters";

                using (SqlCommand command = new SqlCommand(sql, connection))

                {

                    connection.Open();

                    using (SqlDataReader reader = command.ExecuteReader())

                    {

                        while (reader.Read())

                        {                            

                            int x = (byte)reader[0];

                            ListViewItem semester = new ListViewItem(x.ToString());

                            semester.SubItems.Add(reader.GetDateTime(1).ToString("yyyy-MMM-dd"));

                            semester.SubItems.Add(reader.GetDateTime(2).ToString("yyyy-MMM-dd"));

                            lstSemesters.Items.Add(semester);

                        }

                        reader.Close();

                    }

                    connection.Close();

                }

            }

        }
```

The below procedure removes all dates and semester from the database:

```csharp
        private void btnDeleteSem_Click(object sender, EventArgs e)

        {

            if (lstSemesters.SelectedItems.Count != 1)

                MessageBox.Show("You need to select one semester from the list first!");

            else

            {

                DateTime startDate = dtInitial.Value, endDate = dtFinal.Value;

                int semToDelete = int.Parse(lstSemesters.SelectedItems[0].Text);

                String sql;

                string message = "Are you sure you want to remove the semester and all associated data?";

                string title = "Confirmation";

                MessageBoxButtons buttons = MessageBoxButtons.YesNo;

                DialogResult result = MessageBox.Show(message, title, buttons);

                if (result == DialogResult.Yes)

                {

                    using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

                    {

                        sql = "SELECT StartDate, EndDate FROM dbo.Semesters WHERE SemesterNumber = " + semToDelete + ";";

                        using (SqlCommand command = new SqlCommand(sql, connection))

                        {

                            connection.Open();

                            using (SqlDataReader reader = command.ExecuteReader())

                            {

                                while (reader.Read())

                                {

                                    startDate = reader.GetDateTime(0);

                                    endDate = reader.GetDateTime(1);

                                }

                                reader.Close();

                            }

                            connection.Close();

                        }

                    }

                    for (var dt = startDate; dt <= endDate; dt = dt.AddDays(1))

                    {

                        DeleteAbsencesFromDate(dt);

                    }

                    //delete all Absences for these dates

                    sql = "DELETE FROM Dates WHERE SemesterNumber=" + semToDelete + ";";

                    ExecuteSQLInsert(sql);

                    sql = "DELETE FROM Semesters WHERE SemesterNumber=" + semToDelete + ";";

                    ExecuteSQLInsert(sql);

                    UpdateSemList();

                }

            }            

        }

        private void DeleteAbsencesFromDate(DateTime date)

        {

            String sql = "DELETE FROM Attendances WHERE Date='" + date.ToString("yyyy-MM-dd") + "';";

            ExecuteSQLInsert(sql);

        }
```

The below procedure displays all students absent from a given group for the date/period chosen:

```csharp
        private void btnGroupAbsent_Click(object sender, EventArgs e)

        {

            lstAbsences.Items.Clear();

            string group;

            DateTime selectedDT = dtRegistrar.Value;

            int period = Convert.ToInt32(Math.Round(nmPeriod.Value, 0));

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                String sql;

                if (cmbAbsentGroups.Text != null)

                {

                    sql = "SELECT Attendances.StudentID, Students.FirstName, Students.LastName, Students.StudentGroup, Students.MotherName, Students.MotherPhone, Students.FatherName, Students.FatherPhone, Students.ThirdName, Students.ThirdRole, Students.ThirdPhone FROM Attendances " +

                    "INNER JOIN Students ON Attendances.StudentID = Students.StudentID WHERE Date = '" + selectedDT.ToString("yyyy-MM-dd") + "' AND Period = " + period + " AND IsPresent = 0;";

                }

                else

                {

                    group = cmbAbsentGroups.Text;

                    sql = "SELECT Attendances.StudentID, Students.FirstName, Students.LastName, Students.StudentGroup, Students.MotherName, Students.MotherPhone, Students.FatherName, Students.FatherPhone, Students.ThirdName, Students.ThirdRole, Students.ThirdPhone FROM Attendances " +

                    "INNER JOIN Students ON Attendances.StudentID = Students.StudentID WHERE StudentGroup='" + group + "' AND Date = '" + selectedDT.ToString("yyyy-MM-dd") + "' AND Period = " + period + " AND IsPresent = 0;";

                }

                using (SqlCommand command = new SqlCommand(sql, connection))

                {

                    connection.Open();

                    using (SqlDataReader reader = command.ExecuteReader())

                    {

                        while (reader.Read())

                        {

                            ListViewItem student = new ListViewItem(reader.GetInt32(0).ToString());

                            student.SubItems.Add(reader.GetString(1));

                            student.SubItems.Add(reader.GetString(2));

                            student.SubItems.Add(reader.GetString(3));

                            student.SubItems.Add(reader.GetString(4)); //Mother's Name

                            student.SubItems.Add(reader.GetInt32(5).ToString()); //Mother's Phone Number                            

                            student.SubItems.Add(reader.GetString(6)); //Father's Name

                            student.SubItems.Add(reader.GetInt32(7).ToString()); //Father's Phone Number

                            student.SubItems.Add(reader.GetString(8)); //Backup Contact's Name

                            if (!reader.IsDBNull(9))

                                student.SubItems.Add(reader.GetString(9)); //Backup Contact's Role

                            student.SubItems.Add(reader.GetInt32(10).ToString()); //Backup Contact's Phone Number

                            lstAbsences.Items.Add(student);

                        }

                        reader.Close();

                    }

                    connection.Close();

                }

            }

        }
```

The below procedure generates a flat file of all students absent (from a given group) for the date/period chosen:

```csharp
        private void btnFlatGen_Click(object sender, EventArgs e)

        {

            string group;

            DateTime selectedDT = dtRegistrar.Value;

            int period = Convert.ToInt32(Math.Round(nmPeriod.Value, 0));

            List<Student> studentsToCsv = new List<Student>();

            Student tempStudent;

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                String sql;

                if (cmbAbsentGroups.Text != null)

                {

                    sql = "SELECT Attendances.StudentID, Students.FirstName, Students.LastName, Students.StudentGroup, Students.MotherName, Students.MotherPhone, Students.FatherName, Students.FatherPhone, Students.ThirdName, Students.ThirdRole, Students.ThirdPhone FROM Attendances " +

                    "INNER JOIN Students ON Attendances.StudentID = Students.StudentID WHERE Date = '" + selectedDT.ToString("yyyy-MM-dd") + "' AND Period = " + period + " AND IsPresent = 0;";

                }

                else

                {

                    group = cmbAbsentGroups.Text;

                    sql = "SELECT Attendances.StudentID, Students.FirstName, Students.LastName, Students.StudentGroup, Students.MotherName, Students.MotherPhone, Students.FatherName, Students.FatherPhone, Students.ThirdName, Students.ThirdRole, Students.ThirdPhone FROM Attendances " +

                    "INNER JOIN Students ON Attendances.StudentID = Students.StudentID WHERE StudentGroup='" + group + "' AND Date = '" + selectedDT.ToString("yyyy-MM-dd") + "' AND Period = " + period + " AND IsPresent = 0;";

                }

                using (SqlCommand command = new SqlCommand(sql, connection))

                {

                    connection.Open();

                    using (SqlDataReader reader = command.ExecuteReader())

                    {

                        while (reader.Read())

                        {

                            tempStudent = new Student(reader.GetString(1), reader.GetString(2), reader.GetString(3), reader.GetString(4), reader.GetString(6), reader.GetString(8), reader.GetString(9), reader.GetInt32(5), reader.GetInt32(7), reader.GetInt32(10));

                            studentsToCsv.Add(tempStudent);

                        }

                        reader.Close();

                    }

                    connection.Close();

                }

            }

            using (var fbd = new FolderBrowserDialog())

            {

                DialogResult result = fbd.ShowDialog();

                if (result == DialogResult.OK && !string.IsNullOrWhiteSpace(fbd.SelectedPath))

                {

                    using (var writer = new StreamWriter(fbd.SelectedPath + "\\absences-on-" + selectedDT.ToString("yyyy-MM-dd") + ".csv"))

                    using (var csv = new CsvWriter(writer, CultureInfo.InvariantCulture))

                    {

                        csv.WriteRecords(studentsToCsv);

                    }

                }

            }

        }
```

The below procedure removes a user and all their associated records from the database:

```csharp
        private void btnRemoveUser_Click(object sender, EventArgs e)

        {

            //must check if something is selected, then if user is logged in, then if the user is trying to delete themselves

            if (lstUsers.SelectedItems.Count == 1)

            {

                if (frmLogin.passUser == lstUsers.SelectedItems[0].Text)

                {

                    MessageBox.Show("You cannot remove yourself!");

                }

                else

                {

                    if (lstUsers.SelectedItems[0].SubItems[4].Text == "True")

                    {

                        MessageBox.Show("This user is currently logged in. Have them log out first.");

                    }

                    else

                    {

                        string userToDelete = lstUsers.SelectedItems[0].Text;

                        string message = "Are you sure you want to delete student with Username " + userToDelete + "?";

                        string title = "Confirmation";

                        MessageBoxButtons buttons = MessageBoxButtons.YesNo;

                        DialogResult result = MessageBox.Show(message, title, buttons);

                        if (result == DialogResult.Yes)

                        {

                            String sql;

                            if (lstUsers.SelectedItems[0].SubItems[3].Text == "t")

                            {

                                sql = "UPDATE Groups SET TeacherRep = null WHERE TeacherRep = '" + userToDelete + "';"; //Removes teacher as Representative of any other group.

                                ExecuteSQLInsert(sql);

                                sql = "DELETE FROM dbo.Teachings WHERE TeacherUsername = '" + userToDelete + "';";

                                ExecuteSQLInsert(sql);

                                //absences can remain unaffected.

                            }

                            if (frmLogin.passRights == 'a' || frmLogin.passRights == 's')

                            {

                                sql = "DELETE FROM dbo.Feed WHERE Author = '" + userToDelete + "';";

                                ExecuteSQLInsert(sql);

                            }

                            sql = "DELETE FROM dbo.Users WHERE UserName = '" + userToDelete + "';";

                            ExecuteSQLInsert(sql);

                            UpdateUsersList();

                        }

                    }

                }

            }

            else

            {

                MessageBox.Show("You have not selected a user.");

            }

        }
```

The below procedure is used to execute an SQL command:

```csharp
        private void ExecuteSQLInsert(String sqlCommand)

        {

            StringBuilder errorMessages = new StringBuilder();

            using (SqlConnection connection = new SqlConnection(builder.ConnectionString))

            {

                using (SqlCommand command = new SqlCommand(sqlCommand, connection))

                {

                    try

                    {

                        connection.Open();

                        try

                        {

                            command.ExecuteNonQuery();

                        }

                        catch (SqlException ex)

                        {

                            for (int i = 0; i < ex.Errors.Count; i++)

                            {

                                errorMessages.Append("Message: " + ex.Errors[i].Message);

                            }

                            if (errorMessages.ToString().Contains("Violation of PRIMARY KEY") && errorMessages.ToString().Contains("dbo.Lessons"))

                                MessageBox.Show("The database already contains a lesson with this ID. Try entering a new ID");

                            else if (errorMessages.ToString().Contains("Violation of PRIMARY KEY") && errorMessages.ToString().Contains("dbo.Group"))

                                MessageBox.Show("This group already exists.");

                            else

                                MessageBox.Show(errorMessages.ToString());

                        }

                        connection.Close();

                    }

                    catch

                    {

                        MessageBox.Show("Could not reach the database. Check your connection.");

                    }

                }

            }

        }
```

The below procedure removes a student and their associated attendance records from the database:

```csharp
        private void btnRemove_Click(object sender, EventArgs e) //delete student from Students, Absences

        {

            if (lstStudents.SelectedItems.Count == 1)

            {

                string StudentID = lstStudents.SelectedItems[0].Text;

                string message = "Are you sure you want to delete student with ID " + StudentID + "?";

                string title = "Confirmation";

                MessageBoxButtons buttons = MessageBoxButtons.YesNo;

                DialogResult result = MessageBox.Show(message, title, buttons);

                if (result == DialogResult.Yes)

                {

                    String sql = "DELETE FROM dbo.Attendances WHERE StudentID = " + StudentID + ";";

                    ExecuteSQLInsert(sql);

                    sql = "DELETE FROM dbo.Students WHERE StudentID = " + StudentID + ";";

                    ExecuteSQLInsert(sql);

                    lstStudents.SelectedItems[0].Remove();

                }

                else

                {

                    MessageBox.Show("The student will not be deleted.");

                }

            }

            else

                MessageBox.Show("You need to select a student from the list first.");

        }
```

The below procedure removes a group and its associated student, attendance and teaching records from the database:

```csharp
        private void btnRemoveGrpAndStudents_Click(object sender, EventArgs e)

        {

            if (lstStudents.SelectedItems.Count == 1)

            {

                //remove groups and students here

                string GroupID = lstGroups.SelectedItems[0].Text;

                string message = "Are you sure you want to delete the group with ID " + GroupID + "? This will remove all students in that group.";

                string title = "Confirmation";

                MessageBoxButtons buttons = MessageBoxButtons.YesNo;

                DialogResult result = MessageBox.Show(message, title, buttons);

                if (result == DialogResult.Yes)

                {

                    String sql = "DELETE FROM dbo.Attendances WHERE MemberOf = '" + GroupID + "';";

                    ExecuteSQLInsert(sql);

                    sql = "DELETE FROM dbo.Students WHERE StudentGroup = '" + GroupID + "';";

                    ExecuteSQLInsert(sql);

                    lstGroups.SelectedItems[0].Remove();

                }

            }

            else

            {

                MessageBox.Show("You need to select a group from the list first!");

            }

        }

    }
}
```
