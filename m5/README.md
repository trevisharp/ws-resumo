| Artefato                 | Valor |
|--------------------------|-------|
| Entregas                 | 0.8   |
| Guia de Estilo (-Fonte)  | 0.4   |
| Gerenciar de Avaliações  | 1.5   |
| Teste de Unidade         | 2.0   |
| Teste Funcional          | 2.0   |
| Nova Avaliação           | 1.7   |

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

# Testes

```cs
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;
using System.Collections.Generic;
using System.Linq;

namespace UnitTestProject1
{

    public class RiscoService
    {
        public Double NotaFinalPonderada(double[] notas)
        {
            return notas.Average();
        }

        public string NivelRisco(double[] notas)
        {
            var media = NotaFinalPonderada(notas);
            if (media >= 8)
                return "Baixo risco";
            if (media >= 5)
                return "Risco médio";

            return "Riscco alto";
        }
    }

    [TestClass]
    public class UnitTest1
    {
        [TestMethod]
        public void CenarioRiscoBaixo()
        { 
            double[] nota = { 9, 10, 9, 10,10,9,9};
            RiscoService service =  new RiscoService();
            var risco = service.NivelRisco(nota);
            Assert.AreEqual("Baixo risco", risco);
        }


        [TestMethod]
        public void CenarioRiscoMédio()
        {
            double[] nota = { 6,6,7,6,6,7,7 };
            RiscoService service = new RiscoService();
            var risco = service.NivelRisco(nota);
            Assert.AreEqual("Risco médio", risco);
        }

        [TestMethod]
        public void CenarioRiscoAlto()
        {
            double[] nota = { 4,3,2,4,4,4,4 };
            RiscoService service = new RiscoService();
            var risco = service.NivelRisco(nota);
            Assert.AreEqual("Riscco alto", risco);
        }

        [TestMethod]
        public void CenarioRiscoLimitrofe()

        {
            double[] nota = { 4.9,  4.9, 4.9,4.9,4.9,4.9,4.9};
            RiscoService service = new RiscoService();
            var risco = service.NivelRisco(nota);
            Assert.AreEqual("Riscco alto", risco);
        }

        [TestMethod]
        public void CenarioRiscoAltoLimitrofe()
        {
            double[] nota = { 5,5,5,5,5,5,5 };
            RiscoService service = new RiscoService();
            var risco = service.NivelRisco(nota);
            Assert.AreEqual("Risco médio", risco);
        }
    }
}
```