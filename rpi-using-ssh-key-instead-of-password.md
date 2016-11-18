# Raspberry Pi - Using SSH Key instead of password

First you need to generate SSH Keys on your development machine:

```bash
ssh-keygen
```

Then upload your keys to Raspberry Pi by typing following command:

```bash
ssh-copy-id pi@<Raspberry Pi IP Address> 
```

> [!NOTE]
> On Windows you can use Git bash to execute **ssh-keygen** and **ssh-copy-id** commands
