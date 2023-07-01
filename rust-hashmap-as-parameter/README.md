To send a HashMap as a parameter to a Rust function from the `near-cli`, you can use the `--arg` flag to specify the key-value pairs of the map as command-line arguments. The `--arg` flag takes two arguments: the key and the value, separated by a `=` sign.

For example, suppose you have a Rust function named `my_function` that takes a `HashMap<String, String>` as an argument. You could call this function from the `near-cli` using the following command:

```
near call <contract_name> my_function --arg key1=value1 --arg key2=value2

```

This would pass a HashMap with two entries to the `my_function` function, where the keys are "key1" and "key2", and the corresponding values are "value1" and "value2", respectively.

Inside the my_function function, you can access the HashMap argument using the `std::env::args` method, which returns an iterator over the command-line arguments passed to the function. You can then use the `Iterator::filter` method to filter the arguments for those with the `--arg` flag, and the `Iterator::map` method to map each argument to a tuple containing the key and value. Finally, you can use the `Iterator::collect` method to collect the tuples into a HashMap:

```
use std::collections::HashMap;
use std::env;

fn my_function(map: HashMap<String, String>) {
    // function body goes here
}

#[near_bindgen]
impl MyContract {
    pub fn call_my_function() {
        let args: Vec<String> = env::args().collect();
        let map: HashMap<String, String> = args
            .iter()
            .filter(|arg| arg.starts_with("--arg"))
            .map(|arg| arg.split("=").collect::<Vec<&str>>())
            .map(|parts| (parts[0].to_string(), parts[1].to_string()))
            .collect();

        my_function(map);
    }
}

```
I hope this helps! Let me know if you have any other questions.
