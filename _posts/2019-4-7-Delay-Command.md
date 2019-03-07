---
layout: post
title: Komenda z opóźnionym zapłonem
---

Implementacja RelayCommand z opóźnieniem wykonania akcji. Wykonanie komendy można anulować poprzez ponowne wciśnięcie przycisku.

![_config.yml]({{ site.baseurl }}/images/start-stop-engine-button.jpg)



DelayRelayCommand.cs

~~~ csharp
public class DelayRelayCommand : ICommand
    {
        public event EventHandler CanExecuteChanged;

        private readonly Action execute;
        private readonly Func<bool> canExecute;

        private TimeSpan delay;

        CancellationTokenSource cancellationTokenSource = new CancellationTokenSource();

        CancellationToken token;

        private bool isWaiting;

        public DelayRelayCommand(Action execute, Func<bool> canExecute = null, TimeSpan delay = default(TimeSpan))
        {
            this.execute = execute;
            this.canExecute = canExecute;
            this.delay = delay;
            token = cancellationTokenSource.Token;
        }

        public bool CanExecute(object parameter)
        {
            return canExecute == null || canExecute();
        }

        public void Execute(object parameter)
        {   
            if (isWaiting)
            {
                cancellationTokenSource.Cancel();
                return;
            }

            isWaiting = true;
            
            Task.Delay(delay, token)
                .ContinueWith(t=>execute?.Invoke(), TaskContinuationOptions.NotOnCanceled)
                    .ContinueWith(t => isWaiting = false);
            
        }
    }
~~~


ViewModel.cs
~~~ csharp
 public class ShellViewModel : ViewModelBase
{
    public ICommand FireCommand { get; private set; }

    public ShellViewModel()
    {   
        FireCommand = new DelayRelayCommand(() => Fire(), delay: TimeSpan.FromSeconds(5));
    }

    private void Fire()
    {

    }

}
~~~
