# cSharp-winform-guide
A Winform Guide For  my c# Exam

# For your Global variables dont forget the add this lines :
```
 public string connectionString = "Data Source=test.db;Version=3";
string selectedImagePath = "";
```

## Add Database Connection With Datagridview, This add a Contextmenustrip to Form1_Load, Dont use two times Form1_Load func on your project. 
```
private void CheckIfDataExists()
        {
            if (!File.Exists("test.db"))
            {
                SQLiteConnection.CreateFile("test.db");
                using (SQLiteConnection conn = new SQLiteConnection(connectionString))
                {
                    string commandstring = "CREATE TABLE Contacts(Id INTEGER PRIMARY KEY AUTOINCREMENT, Name NVARCHAR(250), Email NVARCHAR(250), ImageUrl NVARCHAR(250))";

                    using (SQLiteCommand cmd = new SQLiteCommand(commandstring, conn))
                    {
                        conn.Open();
                        cmd.ExecuteNonQuery();
                    }
                }
            }
            else
            {
                LoadContactDataGridView();

            }
        }

        private void LoadContactDataGridView()
        {
            dataGridView1.DataSource = GetDataFromDatabase();

        }
        

        private DataTable GetDataFromDatabase()
        {
            DataTable dtContacts = new DataTable();
            using (SQLiteConnection conn = new SQLiteConnection(connectionString))
            {
                string commandstring = "SELECT * FROM Contacts";
                using (SQLiteCommand cmd = new SQLiteCommand(commandstring, conn))
                {
                    conn.Open();
                    SQLiteDataReader reader = cmd.ExecuteReader();

                    dtContacts.Load(reader);

                }
            }

            return dtContacts;
        }
        private void Form1_Load(object sender, EventArgs e)
        {
            CheckIfDataExists();

            // ContextMenuStrip oluştur
            ContextMenuStrip contextMenuStrip = new ContextMenuStrip();
            ToolStripItem deleteItem = contextMenuStrip.Items.Add("Sil");
            deleteItem.Click += DeleteItem_Click;

            // DataGridView'e ContextMenuStrip'i ata
            dataGridView1.ContextMenuStrip = contextMenuStrip;
        }
        // "Sil" menü öğesine tıklandığında çalışacak olay işleyicisi
        private void DeleteItem_Click(object sender, EventArgs e)
        {
            if (dataGridView1.SelectedRows.Count > 0)
            {
                foreach (DataGridViewRow row in dataGridView1.SelectedRows)
                {
                    if (!row.IsNewRow)
                    {
                        // Veritabanından sil
                        DeleteRecordByRow(row);
                        dataGridView1.Rows.Remove(row);
                        
                    }
                }
            }
            else
            {
                MessageBox.Show("Lütfen silmek istediğiniz satırı seçin.");
            }
        }
        private void DeleteRecordByRow(DataGridViewRow row)
        {
            int id = Convert.ToInt32(row.Cells["Id"].Value);

            using (SQLiteConnection conn = new SQLiteConnection(connectionString))
            {
                conn.Open();

                string sql = "DELETE FROM Contacts WHERE Id = @Id"; // SQL sorgusu
                using (SQLiteCommand cmd = new SQLiteCommand(sql, conn))
                {
                    cmd.Parameters.AddWithValue("@Id", id);
                    cmd.ExecuteNonQuery();
                }
            }
        }
```

# değerleri işlemek için btnSave adında bir buton oluşturun, ve ardından bu kodu kopyalayabilirsiniz. Text değerleri formunuza göre değiştirin. Bunun için saveToDatabase ve  getText fonksiyonlarını düzenleyin.
```
 private void btnSave_Click(object sender, EventArgs e)
        {

            
            string text = getText();
            


            imageSave();
            saveToDatabase();
            SaveDataToFile(text);
            MailGonderme(text);

        }

        private void imageSave()
        {
            // Uygulama klasöründe bir 'Images' klasörü oluştur
            string imagesFolderPath = Path.Combine(Application.StartupPath, "Images");
            Directory.CreateDirectory(imagesFolderPath);
            if(selectedImagePath != "")
            {
                // Resmi yeni klasöre kopyala
                string destPath = Path.Combine(imagesFolderPath, Path.GetFileName(selectedImagePath));
                File.Copy(selectedImagePath, destPath, true);
            }
          
        }
        private void saveToDatabase()
        {
            string name = txtName.Text;
            string email = txtEmail.Text;

            string image = Path.GetFileName(selectedImagePath);
            // SQLite bağlantı dizesi
            using (SQLiteConnection conn = new SQLiteConnection(connectionString))
            {
                conn.Open();

                string sql = "INSERT INTO Contacts (Name, Email, ImageUrl) VALUES (@Name, @Email, @ImageUrl)";
                using (SQLiteCommand cmd = new SQLiteCommand(sql, conn))
                {
                    // Burada kullanıcıdan alınan diğer verileri ekleyin
                    // Örneğin:
                    cmd.Parameters.AddWithValue("@Name", name);
                    cmd.Parameters.AddWithValue("@Email", email);
                    cmd.Parameters.AddWithValue("@ImageUrl", image);

                    cmd.ExecuteNonQuery();
                    CheckIfDataExists();
                }
            }
        }
        private void MailGonderme(string message)
        {
            try
            {
                //MailMessage mail = new MailMessage();
                //SmtpClient SmtpServer = new SmtpClient("your_smtp_server");
                //mail.From = new MailAddress("your_email_address");
                //mail.To.Add("recipient_email_address");
                //mail.Subject = "E-posta Konusu";
                //mail.Body = "message";

                //// Eğer SMTP sunucunuz kimlik doğrulaması gerektiriyorsa:
                //SmtpServer.Port = 587; // veya kullanılan porta göre değiştirin
                //SmtpServer.Credentials = new NetworkCredential("username", "password");
                //SmtpServer.EnableSsl = true; // Sunucu SSL gerektiriyorsa

                //SmtpServer.Send(mail);
                MessageBox.Show("E-posta Gönderildi.");

            }
            catch (Exception ex)
            {
                MessageBox.Show("Hata: " + ex.Message);
            }
        }

        private void SaveDataToFile(string message)
        {
            string filePath = "savedData.txt"; // Metin dosyasının yolu ve adı
           

            try
            {
                using (StreamWriter writer = new StreamWriter(filePath, true)) // 'true' dosyaya eklemek için
                {
                    writer.WriteLine(message);
                }

                MessageBox.Show("Veriler dosyaya kaydedildi.");


               
            }
            catch (Exception ex)
            {
                MessageBox.Show("Dosyaya kaydederken hata oluştu: " + ex.Message);
            }
        }
```
