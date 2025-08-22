| Artefato                 | Valor |
|--------------------------|-------|
| Entregas                 | 0.3   |
| Dados Carregados         | 0.4   |
| Design                   | 0.5   |
| Navegabilidade           | 0.5   |
| Login                    | 0.5   |
| Dashboard                | 1.45  |
| Habilitar Botão Exportar | 0.15  |
| Card Transferencia       | 1.4   |
| Tabela Tipo Transporte   | 1.2   |
| Antendimentos - Graficos | 1.2   |
| Atendimentos - Paginação | 1.6   |
| Dashboard - Filtros      | 2.0   |

# Card de Transferências
```cs
public async void GerarTabela()
{
    var db = new Entities1();
    var dados = db.dados_xlsx___TransferenciasPacientes
        .ToArray()
        .Where(u => u.DataTransferencia.Contains($"/{ano}"))
        .GroupBy(u => new {
            Mes = int.Parse(u.DataTransferencia.Split('/')[0]),
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
