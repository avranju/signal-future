# signal-future

A [Tokio](https://tokio.rs/) future implementation that represents a one-time signal future. Example:

```rust
use std::sync::{Arc, Mutex};
use std::time::{Duration, Instant};
use tokio::timer::Delay;

const WAIT_SECS: u64 = 2;

fn main() {
  let mut sf = signal_future::signal();
  let sf2 = sf.clone();

  let signalled = Arc::new(Mutex::new(false));

  let t1 = Delay::new(Instant::now() + Duration::from_secs(WAIT_SECS))
      .map_err(|_| ())
      .map(move |_| sf.signal());

  let signalled_copy = signalled.clone();
  let t2 = sf2.map(move |_| {
      let mut signalled = signalled_copy.lock().unwrap();
      *signalled = true;
  });

  tokio::run(t1.join(t2).map(|_| ()));
  assert_eq!(true, *signalled.lock().unwrap());
}
```