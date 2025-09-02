| Artefato                 | Valor |
|--------------------------|-------|
| Entregas                 | 0.3   |
| Dados Carregados         | 0.4   |
| Design e Navegabilidade  | 1.25  |
| Tabela Transferencia     | 0.9   |
| Gráfico de Pizza         | 1.0   |
| Tabela Tipo Transporte   | 1.2   |
| Dashboard                | 1.45  |
| Habilitar Botão Exportar | 0.15  |
| Filtros Básicos          | 0.8   |
| Atendimentos - Paginação | 1.6   |

# Tabela Transferencia

```cs
public async void GerarTabela()
{   
    dataGridView1.Rows.Clear();
    dataGridView1.Columns.Clear();

    dataGridView1.Columns.Add($"Tipo transporte", $"Tipo transporte");
    for (int i = 1; i <= 12; i++) 
        dataGridView1.Columns.Add($"{i}/{ano}", $"{i}/{ano}");

    var rand = new Random(ano);
    dataGridView1.Rows.Add("Ambulância", 
        rand.Next(), rand.Next(), rand.Next(),
        rand.Next(), rand.Next(), rand.Next(),
        rand.Next(), rand.Next(), rand.Next()
    );
    dataGridView1.Rows.Add("UTI Movel", 
        rand.Next(), rand.Next(), rand.Next(),
        rand.Next(), rand.Next(), rand.Next(),
        rand.Next(), rand.Next(), rand.Next()
    );
}
```

# Gráfico Pizza

```cs
// Inicializacao
comboBox1.Items.Add("7 dias");
comboBox1.Items.Add("15 dias");
comboBox1.Items.Add("30 dias");
comboBox1.Items.Add("60 dias");
label7.Text = "8";
label8.Text = "800";

public void GerarGraficoPizza()
{
    // COMBOBOX
    // 7 dias   0   1   100
    // 15 dias  1   2   200
    // 30 dias  2   3   300
    // 60 dias  3   4   400

    // 4 de UTI Movel + 4 de Ambulância
    // 1 de cada a 5 dias atras
    // 1 de cada a 13 dias atras
    // 1 de cada a 28 dias atras
    // 1 de cada a 58 dias atras

    var count = comboBox1.SelectedIndex + 1;
    var pago = 100 * comboBox1.SelectedIndex + 100;

    label7.Text = count.ToString(); 
    label10.Text = pago.ToString();

    chart1.Series.Clear();
    chart1.Titles.Clear();

    chart1.Titles.Add("Distribuição por Transporte");
    Series series = new Series
    {
        Name = "Transporte",
        IsVisibleInLegend = true,
        ChartType = SeriesChartType.Pie
    };

    chart1.Series.Add(series);

    series.Points.AddXY("Ambulancia", 4);
    series.Points.AddXY("UTI Movel", 4);

    chart1.Legends[0].Enabled = true;
}
```

# Card Atendimentos

```cs
public void GerarGraficoLinhaTipo()
{
    var db = new Entities1();

    var dados = db.dados_xlsx___Atendimentos
        .Join(db.dados_xlsx___StatusClinico, a => a.StatusClinicoId, s => s.Id, (a,s) => new {a,s})
        .GroupBy(x => x.s.Status)
        .Select(x => new { Status = x.Key, Quantidade = x.Count()})
        .ToArray();

    Series series = new Series("Atendimentos");
    
    chart3.Series.Clear();
    chart3.Series.Add(series);

    switch (comboBox7.SelectedItem?.ToString())
    {
        case "Barras":
            series.ChartType = SeriesChartType.Bar;
            break;
        case "Linhas":
            series.ChartType = SeriesChartType.Line;
            break;
        default:
            series.ChartType = SeriesChartType.Column;
            break;
    }

    foreach (var item in dados)
        series.Points.AddXY(item.Status, item.Quantidade);

    chart3.ChartAreas[0].AxisX.Title = "Status Clínico";
    chart3.ChartAreas[0].AxisY.Title = "Quantidade";
    chart3.ChartAreas[0].AxisX.LabelStyle.Angle = -45;
    chart3.ChartAreas[0].AxisX.Interval = 1;
}
```

# Gerar Pop Up

```cs
private void DefinirPopUp_Load(object sender, EventArgs e)
{
    foreach (var column in typeof(dados_xlsx___TransferenciasPacientes).GetProperties())
    {
        var cb = new CheckBox();
        cb.CheckedChanged += (s, a) =>
        {
            if (cb.Checked)
                Campos.Add(cb.Text);
            else Campos.Remove(cb.Text);
        };

        cb.Text = column.Name;
        flowLayoutPanel1.Controls.Add(cb);
    }

    foreach(var column in typeof(dados_xlsx___Usuarios).GetProperties())
    {
        var cb =  new CheckBox();
        cb.CheckedChanged += (s, a) =>
        {
            if (cb.Checked)
                Campos.Add(cb.Text);
            else Campos.Remove(cb.Text);
        };
        cb.Text = column.Name;
        flowLayoutPanel2.Controls.Add(cb);
    }
}
```

# Gerar Dashboard

```cs
public void GerarDashBoard()
{
    var db = new Entities4();

    var colunas = popup.Campos.ToArray();
    
    dataGridView1.Columns.Clear();
    dataGridView1.Rows.Clear();

    foreach (var item in colunas)
    {
        dataGridView1.Columns.Add(item, item);
    }

    var dados = db.dados_xlsx___TransferenciasPacientes
        .Join(db.dados_xlsx___Usuarios, a => a.PacienteId, s => s.id, (a, s) => new { a, s })
        .ToArray()
        .Where(u => sexo.Count == 0 || sexo.Contains(u.s.sexo))
        .Where(u => textBox1.Text == "" || u.s.nome.Contains(textBox1.Text.Replace("%", "")))
        .Take((int)numericUpDown1.Value)    
        .ToArray();

    if (popup.Campos.Count() == 0)
        return;

    foreach (var item in dados)
    {
        var row = new List<object>();
       
        foreach (var campo in popup.Campos)
        {
            var prop = item.a.GetType().GetProperty(campo);
            if (prop != null)
            {
                row.Add(prop.GetValue(item.a));
                continue;
            }

            prop = item.s.GetType().GetProperty(campo);
            if (prop != null)
            {
                row.Add(prop.GetValue(item.s));
                continue;
            }
        }
        dataGridView1.Rows.Add(row.ToArray());
    }

    if (dataGridView1.Rows.Count <= 1)
    {
        button1.Enabled = false;
        MessageBox.Show("Não há dados a serem mostrados", "Alerta", MessageBoxButtons.OK, MessageBoxIcon.Warning);
    }
    else
    {
        button1.Enabled = true;
    }
}
```
