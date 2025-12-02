# form
подключение к бд

#############
internal class DbCarRepairWorkshop 
{
    public static NpgsqlConnection GetConnection() 
    {
        string connString = "Host=localhost;Port=5432;Username=postgres;Password=ваш_пароль;Database=car_repair_workshop";
        var conn = new NpgsqlConnection(connString);
        try{
          conn.Open();
        }
        catch(NpgsqlException ex){
          MessageBox.Show("MySQL Connection" + ex.Message, "Error", MessageBoxButtons.OK, MessageBoxIcon.Error);
        }
        return conn;
    }
}
####################
главная с карточками
public partial class Form2 : Form {
    public Form2() {
        InitializeComponent();
        LoadMalfunctions();
    }
        private void AddMalfunctionsCard(int id, string name, decimal cost) {
            Panel malfunctionsPanel = new Panel {
                BorderStyle = BorderStyle.FixedSingle,
                Size = new Size(400, 100),
                Margin = new Padding(10),
                Tag = id
            };

            Label lblName = new Label {
                Text = name,
                Font = new Font("Segoe UI", 12, FontStyle.Bold),
                Location = new Point(10, 10),
                AutoSize = true
            };
            Label lblCost = new Label {
                Text = $"Цена: {cost.ToString()} руб.",
                Location = new Point(10, 35),
                AutoSize = true
            };
            Button btnDelete = new Button {
                Text = "Удалить",
                Location = new Point(300, 35),
                Size = new Size(80, 30)
            };
            btnDelete.Click += (s, e) => DeleteMalfunction(id);

        Button btnEdit = new Button {
                Text = "Редактировать",
                Location = new Point(10, 70),
                Size = new Size(120, 25),
                Tag = id
            };
            btnEdit.Click += BtnEditMalfunction_Click;

        malfunctionsPanel.Controls.Add(lblName);
            malfunctionsPanel.Controls.Add(lblCost);
            malfunctionsPanel.Controls.Add(btnDelete);
            malfunctionsPanel.Controls.Add(btnEdit);

            flowLayoutPanel1.Controls.Add(malfunctionsPanel);
        }

        private void BtnEditMalfunction_Click(object sender, EventArgs e) {
        Button btn = (Button)sender;
        int malfunctionId = Convert.ToInt32(btn.Tag);

        EditMalfunctionForm form = new EditMalfunctionForm(malfunctionId);
        form.ShowDialog();

        flowLayoutPanel1.Controls.Clear();
        LoadMalfunctions();
        }

    private void DeleteMalfunction(int id) {
        using (NpgsqlConnection conn = DbCarRepairWorkshop.GetConnection()) {
            string query = "DELETE FROM malfunctions WHERE malfunction_id = @id";
            using (NpgsqlCommand cmd = new NpgsqlCommand(query, conn)) {
                cmd.Parameters.AddWithValue("@id", id);

                try {
                    cmd.ExecuteNonQuery();
                    MessageBox.Show("Запись удалена!");
                }
                catch (Exception ex) {
                    MessageBox.Show("Ошибка удаления: " + ex.Message);
                }
            }
        }

        // обновим список
        flowLayoutPanel1.Controls.Clear();
        LoadMalfunctions();
    }

    private void LoadMalfunctions() {
        using (NpgsqlConnection conn = DbCarRepairWorkshop.GetConnection()) { 
        string query = "SELECT malfunction_id, name, cost_of_work FROM malfunctions";
        try {
                using (NpgsqlCommand cmd = new NpgsqlCommand(query, conn))
                using (NpgsqlDataReader reader = cmd.ExecuteReader()) {
                    while (reader.Read()) {
                        int id = Convert.ToInt32(reader["malfunction_id"]);
                        string name = reader["name"].ToString();
                        decimal cost = Convert.ToDecimal(reader["cost_of_work"]);

                        AddMalfunctionsCard(id, name, cost);
                    }
                }
            }
        catch (Exception ex) {
            MessageBox.Show("Ошибка загрузки данных: " + ex.Message);
        }
        }
        }


    private void BtnAddNew_Click(object sender, EventArgs e) {
        AddMalfunctionForm addForm = new AddMalfunctionForm();
        addForm.ShowDialog();

        // Если в форме действительно добавили данные — обновляем список
        if (addForm.DataAdded) {
            flowLayoutPanel1.Controls.Clear();
            LoadMalfunctions();
        }
    }

    private void Form1_Load(object sender, EventArgs e) {
        LoadMalfunctions();
        BtnAddNew.Click += BtnAddNew_Click;


##################
форма добавления
public partial class AddMalfunctionForm : Form {
    public bool DataAdded { get; private set; } = false;

    public AddMalfunctionForm() {
        InitializeComponent();
    }

    private void btnSave_Click(object sender, EventArgs e) {
        string name = txtName.Text.Trim();
        string costText = txtCost.Text.Trim();

        if (string.IsNullOrEmpty(name) || string.IsNullOrEmpty(costText)) {
            MessageBox.Show("Заполните все поля!");
            return;
        }

        if (!decimal.TryParse(costText, out decimal cost)) {
            MessageBox.Show("Стоимость должна быть числом!");
            return;
        }

        using (NpgsqlConnection conn = DbCarRepairWorkshop.GetConnection()) {
            string query = "INSERT INTO malfunctions (name, cost_of_work) VALUES (@name, @cost)";
            using (NpgsqlCommand cmd = new NpgsqlCommand(query, conn)) {
                cmd.Parameters.AddWithValue("@name", name);
                cmd.Parameters.AddWithValue("@cost", cost);

                try {
                    cmd.ExecuteNonQuery();
                    MessageBox.Show("Неисправность добавлена!");
                    DataAdded = true;
                    this.Close(); // Закрываем форму
                }
                catch (Exception ex) {
                    MessageBox.Show("Ошибка при добавлении: " + ex.Message);
                }
            }
        }
    }
}
    }
}
}

############
форма редактирования
public partial class EditMalfunctionForm : Form {
    private int malfunctionId;

    public EditMalfunctionForm(int id) {
        InitializeComponent();
        malfunctionId = id;
        LoadMalfunctionData();
    }

    private void LoadMalfunctionData() {
        using (NpgsqlConnection conn = DbCarRepairWorkshop.GetConnection()) {
            string query = "SELECT name, cost_of_work FROM malfunctions WHERE malfunction_id = @id";
            using (NpgsqlCommand cmd = new NpgsqlCommand(query, conn)) {
                cmd.Parameters.AddWithValue("@id", malfunctionId);

                using (NpgsqlDataReader reader = cmd.ExecuteReader()) {
                    if (reader.Read()) {
                        txtName.Text = reader["name"].ToString();
                        txtCost.Text = reader["cost_of_work"].ToString();
                    }
                }
            }
        }
    }

    private void btnSave_Click(object sender, EventArgs e) {
        string name = txtName.Text.Trim();
        if (!decimal.TryParse(txtCost.Text.Trim(), out decimal cost)) {
            MessageBox.Show("Введите корректную стоимость!");
            return;
        }

        using (NpgsqlConnection conn = DbCarRepairWorkshop.GetConnection()) {
            string query = "UPDATE malfunctions SET name = @name, cost_of_work = @cost WHERE malfunction_id = @id";
            using (NpgsqlCommand cmd = new NpgsqlCommand(query, conn)) {
                cmd.Parameters.AddWithValue("@name", name);
                cmd.Parameters.AddWithValue("@cost", cost);
                cmd.Parameters.AddWithValue("@id", malfunctionId);

                cmd.ExecuteNonQuery();
                MessageBox.Show("Данные успешно обновлены!");
                this.Close();
            }
        }
    }
}
