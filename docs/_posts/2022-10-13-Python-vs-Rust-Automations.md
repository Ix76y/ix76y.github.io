---
layout: post
title: "Automations - Python vs. Rust"
---
# Which Programming Language to use for Automations

In Cyber Security automations play an important role and can make a hugh difference when it comes down to how much time an analyst needs to spend on a case. Most automations are written in Python, due to the fact that it is easy to learn and write and it has a big library of existing packages that can make life a lot easier when writing automations. 

However, one of the biggest downsides of Python is always said to be its speed. While there are python packages that are written in a different language to speed it up, python still has the reputation of being slow, especially in comparison to other languages like Rust. 

## Why Rust could be the new programming language for automations
Rust is a relatively new language and is generally known to be very secure and fast. Especially Rusts type safety can be a big advantage over Pythons loose way of handling types and could save some trouble down the line. But the bigger question is, is Rust really faster then Python? 

## Rust vs. Python - Speed Test
Testing the speed can be very depending on the task that is being executed and the libraries that are used. With this in mind this test was just checking the speed of both languages for a very specific task, for other tests the result may look very different.

### The Test
As a test, both languages have to read a file with compromised credentials, extract the domain of the email address and then create some simple statistics with how many times a certian domain appeared. 

The credential files have the following pattern:
```
email@domain:password
```

### Python
Python doesn't need any special imports to be able to read a file and split each line to extract the domain. While there are a lot of libraries that could have helped with this task this test was mostly concentrating on using the easiest way without too many external packages.

```python
# Reading the file
f = open("comped_creds.txt", "r")
lines = f.readlines()

# Contains all domains encountered and how often
domains_total = {}
# Contains only the domains that had a password
domains = {}
no_password = 0

for line in lines:
    line = line.strip().split(':')
    # Extract the domain
    domain = line[0].split('@')[1]

    # Count the number of times a domain appears
    domains_total.setdefault(domain, 0) 
    domains_total[domain] += 1

    # Check if there was a password
    if len(line[1]) == 0:
        no_password += 1
    else: 
        domains.setdefault(domain, 0) 
        domains[domain] += 1
```

After reading the file and adding the information to different dictionaries all that was left was to sort them to find the domains that appear most often:

```python
# Sort domains
sorted_domains = dict(sorted(domains.items(), key=lambda x:x[1]))
sorted_domains_total = dict(sorted(domains_total.items(), key=lambda x:x[1]))

n = 5
top_n_domains = list(sorted_domains.keys())[-n: ]

print(f'Number of domains with passwords vs all: {len(domains)}/{len(domains_total)}')
print(f'Top {n} Domains:')
for top_domain in top_n_domains:
    print(f'\t{top_domain}: \t{domains[top_domain]}')
print(f'Number of times with no passwords: {no_password}')
```

With comments and empty lines the full script was **47 lines** in total. 

### Rust

Similar to the Python implementation, Rust was also relying only on functions from the standard library and following a very similar pattern:
```rust
use::std::time::{SystemTime};
use std::fs;
use std::collections::HashMap;

fn main() {
    let path = "comped_creds.txt";
    let content = match fs::read_to_string(&path) {
        Ok(content) => content,
        Err(error) => {
            println!("Coudln't read file: {:?}", error);
            return;
        },
    };

    // Contains all domains encountered and how often
    let mut domains_total: HashMap<&str, u32> = HashMap::new();
    // Contains only the domains that had a password
    let mut domains: HashMap<&str, u32> = HashMap::new();
    let mut no_password = 0;

    for line in content.lines() {
        let line_split: Vec<&str> = line.trim().split(':').collect();
        let domain = line_split[0].split('@').collect::<Vec<&str>>()[1];

        *domains_total.entry(domain).or_insert(0) += 1; 
        
        if line_split[1].len() == 0 {
            no_password += 1;
        } else {
            *domains.entry(domain).or_insert(0) += 1;
        }
    }
    // ...
}
```
And after extracting all the information it was sorted similar to the Python code:
```rust
 // Sort domains
let mut sorted_domains: Vec<(&&str, &u32)> = domains.iter().collect();
sorted_domains.sort_by(|a, b| a.1.cmp(b.1).reverse());

let mut sorted_domains_total: Vec<(&&str, &u32)> = domains_total.iter().collect();
sorted_domains_total.sort_by(|a, b| a.1.cmp(b.1).reverse());

let n = 5;
let (top_n_domains, _) = sorted_domains.split_at(n);
println!("Number of domains with passwords vs all: {}/{}", domains.len(), domains_total.len());
println!("Top {n} Domains: ");
for top_domain in top_n_domains {
    println!("\t{}: \t{}", top_domain.0, top_domain.1);
}
println!("Number of times with no passwords: {no_password}");
```

All in all, with comments and empty lines the full script was **58 lines** in total making it 11 lines longer then the same Python script. 

## Speed Python vs. Rust

Now the only question remaining is the actual speed between Python and Rust. The test was run twice once with a smaller and another time with a larger file. 

| File | Python | Rust |
|------|--------|------|
| .txt with 24613 lines | 22ms  | 114ms  |
|.txt with 608750 lines | 486ms | 2108ms |

The differences are massive but in favor of Python. Rust was nearly 5-times slower. The main reason for this is that Rust was run in Debug mode, which means it doesn't use any optimizations and is in gernal slow to give the developer more information while developing. The following table shows the result after running the Rust program in the release version.

| File | Python | Rust |
|------|--------|------|
| .txt with 24613 lines | 22ms  | 7ms  |
|.txt with 608750 lines | 486ms | 168ms |

After running the relase version of the Rust code it is a lot faster and around three times fater then the Python code. 

## Conclusion
The final question that remains is, is it worth writing automations in Rust instead of Python. It probably depends, while there is definitely a speed increase that could save some time for scripts that take up a lot of time. On the other side, some automations don't depend on speed, for those the more important question is what language is easier to write for the person working on it, where Python might have an advantage.