#Lesson 2: Aiming your artillery
Goal: create a script and test your own service endpoint

###Step 1: Create a simple load script with a short duration and small load (<25 TPS).

```
$ slsart script --endpoint <complete target URL here> --duration 10 --rate 5 --rampTo 10
$ slsart invoke
```

###Step 2: Let's take a look at the script that was created.

```
$ ls
$ <your fav editor> script.yml
```

This is the file you will modify to describe your load test - the script feature is used only to quickly create a basic script for you to run from.  Changing your script does not require a re-deploy, just invoke again with whichever script you desire.

The load script determines what targets are load tested and how.  Artillery.io's configuration script is fully documented on their web page - https://artillery.io/docs/.  Lots of examples can be found at https://github.com/shoreditch-ops/artillery-core/tree/master/test/scripts.  *You may find you need to ensure that duration * rampTo * 5 is divisible by 4 and that rampTo divides 1000 cleanly, as there are currently some floating-point math issues with one of the artillery modules.  See* https://github.com/shoreditch-ops/artillery/issues/243.

Before we do too much customization, let's take a look at cloudwatch and setup reporting to InfluxDB in Lessons 3 and 4.
