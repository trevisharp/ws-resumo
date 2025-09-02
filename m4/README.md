| Artefato                 | Valor |
|--------------------------|-------|
| Entregas                 | 0.3   |
| Dados Carregados         | 0.4   |
| Design e Navegabilidade  | 1.25  |
| Card Transferencia       | 0.9   |
| Antendimentos - Graficos | 1.2   |
| Tabela Tipo Transporte   | 1.2   |
| Dashboard                | 1.45  |
| Habilitar Botão Exportar | 0.15  |
| Filtros Básicos          | 0.8   |
| Atendimentos - Paginação | 1.6   |

# Card de Transferências

Com valores aleatórios:
```cs
public async void GerarTabela()
{   
    dataGridView1.Rows.Clear();
    dataGridView1.Columns.Clear();

    dataGridView1.Columns.Add($"Tipo transporte", $"Tipo transporte");
    for (int i = 0; i < 12; i++) 
        dataGridView1.Columns.Add($"{i+1}/{ano}", $"{i + 1}/{ano}");

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

Com query:

```cs
public async void GerarTabela()
{
    var db = new Entities1();
    var dados = db.dados_xlsx___TransferenciasPacientes
        .ToArray()
        .Where(u => u.DataTransferencia.Contains($"/{ano}"))
        .GroupBy(u => new {
            Mes = int.Parse(u.DataTransferencia.Split('/')[1]),
            u.TipoTransporte
        })
        .Select(g => new {
            g.Key.TipoTransporte,
            g.Key.Mes,
            ValorTransferencias = g.Average(x => x.ValorTotalPago)
        })
       .ToArray();
    
    
    dataGridView1.Rows.Clear();
    dataGridView1.Columns.Clear();

    dataGridView1.Columns.Add($"Tipo transporte", $"Tipo transporte");
    for (int i = 0; i < 12; i++) 
        dataGridView1.Columns.Add($"{i+1}/{ano}", $"{i + 1}/{ano}");

    foreach (var item in dados.GroupBy(j => j.TipoTransporte))
    {
        dataGridView1.Rows.Add(item.Key, 
            item.FirstOrDefault(i => i.Mes == 1)?.ValorTransferencias ?? 0,
            item.FirstOrDefault(i => i.Mes == 2)?.ValorTransferencias ?? 0,
            item.FirstOrDefault(i => i.Mes == 3)?.ValorTransferencias ?? 0,
            item.FirstOrDefault(i => i.Mes == 4)?.ValorTransferencias ?? 0,
            item.FirstOrDefault(i => i.Mes == 5)?.ValorTransferencias ?? 0,
            item.FirstOrDefault(i => i.Mes == 6)?.ValorTransferencias ?? 0,
            item.FirstOrDefault(i => i.Mes == 7)?.ValorTransferencias ?? 0,
            item.FirstOrDefault(i => i.Mes == 8)?.ValorTransferencias ?? 0,
            item.FirstOrDefault(i => i.Mes == 9)?.ValorTransferencias ?? 0,
            item.FirstOrDefault(i => i.Mes == 10)?.ValorTransferencias ?? 0,
            item.FirstOrDefault(i => i.Mes == 11)?.ValorTransferencias ?? 0,
            item.FirstOrDefault(i => i.Mes == 12)?.ValorTransferencias ?? 0
        );
    }
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

# Gráfico Pizza

```cs
public void GerarGraficoPizza()
{
    var db = new Entities4();

    int dias =
        comboBox4.SelectedIndex == -1 ? 
        -1 : int.Parse(comboBox4.Text.Split(' ')[0]);
    string tipo = comboBox5.Text;

    var dados = db.dados_xlsx___TransferenciasPacientes
           .AsEnumerable()
           .Where(i =>
           {
               var partes = i.DataTransferencia.Split('/');
               var dia = int.Parse(partes[1]);
               var mes = int.Parse(partes[0]);
               var ano = int.Parse(partes[2]);

               if (dias == -1)
                   return true;

               if (mes > 9 && ano != 2025 && dia >= 4)
                   return false;

               return (mes >= 7 && dias == 60) ||
                    (mes >= 8 && dias == 30) ||
                    (mes >= 8 && dia >= 20 && dias == 15) ||
                    ((mes == 8 && dia >= 28) || mes == 9 && dias == 7);
           })
           .GroupBy(u => u.TipoTransporte)
           .Select(c => new { Tipo = c.Key, Valores = c.Sum(u => u.ValorTotalPago)  , Quantidade = c.Count()})
           .ToArray();

    int totalqtd = dados
           .Where(u => u.Tipo == tipo || tipo == "")
           .Sum(x => x.Quantidade);
    int valortoal = dados
           .Where(u => u.Tipo == tipo || tipo == "")
           .Sum(x => (int)x.Valores);

    label7.Text = valortoal.ToString();
    label10.Text = totalqtd.ToString();

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

    foreach (var item in dados)
        series.Points.AddXY(item.Tipo, item.Quantidade);

    chart1.Legends[0].Enabled = true;
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
