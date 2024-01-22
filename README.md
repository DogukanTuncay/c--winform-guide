# cSharp-winform-guide
A Winform Guide For  my c# Exam
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
