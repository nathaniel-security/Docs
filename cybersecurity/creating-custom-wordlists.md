# Creating Custom wordlists

### CeWL

* CeWL
  * generated wordlist by crawling website
    * [https://github.com/digininja/CeWL](https://github.com/digininja/CeWL)

```bash
cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
```

### crunch

* crunch
  * [https://secf00tprint.github.io/blog/passwords/crunch/advanced/en](https://secf00tprint.github.io/blog/passwords/crunch/advanced/en)

### Creating custom Usernames

* [https://github.com/urbanadventurer/username-anarchy](https://github.com/urbanadventurer/username-anarchy)

```
git clone https://github.com/urbanadventurer/username-anarchy.git
cd username-anarchy/
```

```
./username-anarchy nathaniel fernandes
```

#### You know the username format and names of users

```
./username-anarchy --input-file ./test-names.txt  --select-format first.last
```

```
andrew.horton
jim.vongrippenvud
peter.otoole
```



### Hashcat

[#generating-rule-based-wordlist](hashcat.md#generating-rule-based-wordlist "mention")



### cupp

* build custom wordlist for that person

```
sudo apt install cupp
```

```shell-session
cupp -i
```
