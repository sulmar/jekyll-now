---
layout: post
title: Komenda z opóźnionym zapłonem
---

Implementacja RelayCommand z opóźnieniem wykonania akcji.

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.

``` csharp
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
```