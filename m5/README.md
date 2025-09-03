| Artefato                 | Valor |
|--------------------------|-------|
| Entregas                 | 0.8   |
| Guia de Estilo (-Fonte)  | 0.4   |
| Gerenciar de Avaliações  | 2.3   |
| Nova Avaliação           | 1.7   |
| Teste de Unidade         | 2.0   |
| Teste Funcional          | 2.0   |

# Deletar Multiplos

```cs
KeyPreview = true;
```

```cs
private void Form1_KeyDown(object sender, KeyEventArgs e)
{
    var db = new Banco();

    if (e.KeyCode == Keys.Delete)
    {
        DialogResult d = MessageBox.Show("Tem certeza da exclusão?", "Aviso", 
            MessageBoxButtons.YesNo, MessageBoxIcon.Question);

        if (d == DialogResult.No)
            return;
                
        foreach (DataGridViewRow item in dataGridView2.SelectedRows)
        {
            int id = (int)item.Cells[0].Value;
            var avalia = db.AvaliacaoRisco.FirstOrDefault(x => x.Id == id);
            db.AvaliacaoRisco.Remove(avalia);
            Deletados.deletados.Add(avalia);
        }

        db.SaveChanges();
        CarregaDataDrid();
    }

    if (e.KeyCode == Keys.Z && e.Control)
    {
        foreach (var item in Deletados.deletados)
            db.AvaliacaoRisco.Add(item);
        Deletados.deletados.Clear();
        
        db.SaveChanges();
        CarregaDataDrid();
    }
}
```

# Caso de Teste

<img src="../imgs/caso-de-teste.jpg" />