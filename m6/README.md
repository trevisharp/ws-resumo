| Artefato                 | Valor |
|--------------------------|-------|
| Entregas + API           | 0.7   |
| Telas + Navegabilidade   | 1.3   |
| Splash                   | 0.7   |
| Validação de Data/Hora   | 0.3   |
| Calendário Básico        | 0.5   |
| Estrelas                 | 0.5   |

# Uteis

```cs
public partial class App : Application
{
    public App()
    {
        InitializeComponent();
        MainPage = new NavigationPage(Splash());
    }

    ContentPage Splash()
    {
        Dispatcher.StartTimer(TimeSpan.FromSeconds(5), () =>
        {
            MainPage.Navigation.PushAsync(Home());
            return false;
        });

        var progress =  new ProgressBar();
        var page = new ContentPage
        {
            Content = new VerticalStackLayout
            {
                Children =
                {
                    new Label { Text = "Spllash" },
                    progress
                },
            }
        };

        page.Loaded += async (sender, e) =>
        {
            await progress.ProgressTo(1,5000,Easing.Linear);
        };

        return page;
    }


    ContentPage Home()
    {
        return new ContentPage
        {
            Content = new VerticalStackLayout
            {
                Children =
                {
                    new Label { Text = "Home" },
                }
            }
        };
    }
}
```