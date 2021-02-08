# DPIE OpenFISCA Wiki

## Welcome

Welcome to the DPIE Energy Security Safeguard OpenFISCA Wiki!

This is a policy maker's personal wiki for implementing Rules as Code for the Energy Security Safeguard, using OpenFISCA. It is intended for a policy maker who has never written Python or even code before, and goes into detail about how to do so.

This repository is a work in progress.

I wrote this in [Hundred Rabbits' Left.](https://100r.co/site/left.html) They do AWESOME work. Check them out.

## What is OpenFISCA?

OpenFISCA is a open-source platform used to write rules as code. It was initially developed by the French government for codifying their taxation systems - but it's now been used for a variety of 

We are one of the first organisations/departments to use it for purposes other than calculating tax - we use it to codify the Energy Savings Scheme Rule, and intend to use it to code the Energy Security Safeguard rules (incl. both the energy efficiency and peak demand response sides of the rules.) 

OpenFISCA is built off Python and thus can be used for the deployment of web apps and services much like vanilla Python, Numpy, Pandas, BeautifulSoup etc. Of course, it can also integrate functions in from other Python libraries (and other languages, if you're good enough to do so.)

More info on OpenFISCA can be found [here.](https://openfisca.org/)

The NSW Department of Customer Service has currently delivered a live community gaming instance of rules as code, using OpenFISCA. [Read more here.](https://www.nsw.gov.au/media-releases/digitising-rules-of-government-to-make-compliance-easy)

## Installation of OpenFISCA

I wrote up a beginner's guide for installing OpenFISCA on your machine, using Docker Desktop, last year. You can find it [here.](https://github.com/liamdmccann/openfisca_windows_tutorial) 

This is most useful if you're going to be a regular user of OpenFISCA. 

If you're going to be an occasional user, or mostly want to experiment, you might find it more useful to use repl.it - it's a lightweight, browser based platform that can show off the power of rules as code without the technical commitment. 

I wrote up a guide for how to use repl.it for OpenFISCA too! You can find it [here.](https://github.com/liamdmccann/openfisca_windows_tutorial)

# Variables 

## What is a variable?

Variables are the building blocks of your legislation. They can be user inputs or calculated by formulas. 

It's easiest to explain with examples. Let's go!

```
class baseline_electricity_consumption(Variable):
    value_type = float
    entity = Building
    definition_period = ETERNITY
    label = "Electricity consumption of the baseline equipment."
```

Let's break down what this means, line by line.

class baseline_electricity_consumption(Variable) means that it's telling Python that this is a class function (essentially a function which bundles data and functionality), it's called baseline_electricity_consumption, and it's using the Variable class defined within OpenFISCA. With one exception (detailed in the Enum section) the only change you'll typically make to this line is the name of the variable.

value_type tells you what type of value will get returned by this variable. choices include:

- bool - a boolean value (True/False)
- date - a date value (this is a datetime object, represented as YYYY-MM-DD)
- Enum - a value from a Enumeration (essentially a set list of values - more on this in the Enum section)
- float - a floating point number (i.e. a number w. a decimal point - 14.0 for instance)
- int - an integer
- str - a text string

entity = Building means what entity the variable is applied to - OpenFISCA can model multiple different entities impacted by a piece of legislation. For example, the French taxation system modelled in OpenFISCA has person (a person), child (a child under 18yo) and a family - and so certain variables are applied to certain groups. 

For the purpose of the ESS everything is being applied to the entity Building, to represent the implementation is taking place. In the longer term this will probably get changed to something like Implementation - this is on the to-do list.

definition_period defines the period for which the variable is applied. This can be MONTH - once a month (meaning one period from 01 to 30/31 of a month), YEAR - once a year or INFINITY - once.

## Enumerables

coming soon. Enumerables are cool and easy, but they take a little bit of time to explain - and you won't need them for Thursdays' workshop. (or will you...)

## Control structures

### Simple logical structures

If you've written anything in Python in the past, you're probably familiar with using syntax like the below:

```
NOTE THIS IS NOT A VALID OPENFISCA FORMULA
def formula(buildings, period, parameters):
    energy_savings = buildings('energy_savings', period)
    if salary < 1000:
        return 200
    else:
        return 0
```

This won't work in OpenFISCA. The reason why it won't work is because OpenFISCA is built on arrays, rather than scalar values. So, my understanding is you can throw a file with 100,000 test cases at OpenFISCA, and it'll calculate the result (almost) as fast as if it was calculating a single test. 

However, this changes how you write your logical structures. 

The way you'd write the above formula, is something like this:

```
def formula(buildings, period, parameters):
    condition_energy_savings = buildings('energy_savings', period) < 1000
    return energy_savings * 200
```

What happens in this case is for every case where condition_energy_savings is True (i.e. the value for energy_savings is less than 1000), it'll assign the value 1 to condition_energy_savings. If it's False (i.e. more than 1000), it'll assign the value 0 to condition_energy_savings. 

This means you can then use the Numpy function where to handle these test cases:

```
def formula(buildings, period, parameters):
    condition_energy_savings = buildings('energy_savings', period) < 1000
    return where(condition_energy_savings, 1000, 200)
```

So in this example, if condition_energy_savings is True, it'll return the value 1000 - if it's False, it'll return 200. Easy!

Of course, you can do lots of interesting things with these condition structures. Below is a short list, demoed through examples:

- condition_is_gas_saving_activity = (gas_saved_more_than_zero * activity_uses_gas) - using * in this formula means that both gas_saved_more_than_zero AND activity_uses_gas must be True for is_gas_saving_activity to be True (you can't multiply by zero)

- condition_has_facial_hair = (has_moustache + has_beard) - using + in this formula means that either has_moustache or has_beard must be true for the condition has_facial_hair to be true 

- condition is_in_europe = not(is_in_america) - you can use not() to return a negation of a value. If a location is in America it can't be in Europe, so if is_in_america is True, is_in_europe will always be False. (US Army Bases don't count.) 

### Complex logical structures

What happens if you have multiple conditions that you want to test for - for example, you want to give a different value to a variable depending on whether a person plays tennis, plays football, plays both, or plays neither? Like, 200 if they only play tennis, 500 if they only play football, 2000 if they play both, and 3000 if they play neither?

Within Python you would use an and function, like this:

```
if plays_football = True and plays_tennis = True: 
  etc etc
if plays_football = True and plays_tennis = False: 
  etc etc
if plays_football = False and plays_tennis = True:
  etc etc
```

Of course, as previously discussed, this won't work - it'll return an error because OpenFISCA is built on arrays. 

Instead, you'd use numpy's select function to implement this. It'd look something like below, excuse the bracketting and formatting: 

```
plays_football = buildings('plays_football', period)
plays_tennis = buildings('plays_tennis', period)
return select([(plays_football * not(plays_tennis)),
               (not(plays_football) * plays_tennis),
               (plays_football * plays_tennis), 
               (not(plays_football) * not(plays_tennis)],
               [200, 500, 2000, 3000])
```

Of course, your number of conditions in the first set of brackets has to match the number of values in the second set of brackets, otherwise it'll get confused and return an error. 

# Parameters

Parameters could be considered the constants of your legislation - values which generally don't change across applications of this legislation. 

## Simple parameters

For example, within the context of the Energy Savings Scheme, all of the Electricity Savings within an implementation are multiplied by 1.06 to get the number of Energy Savings Certificates for an implementation. In OpenFISCA, you can represent this easily in a parameter - see below!

```
description: The Electricity Certificate Conversion Factor, applied to the Electricity Savings
  created through any Recognised Energy Savings Activity.
reference: Equation 1 of the ESS Rule 2020, located in Clause 6.5 of the Energy
  Savings Scheme Rule of 2009. Refers to the Section 130(1) of the Electricity
  Supply Act 1995.
values:
  2020-01-01:
    value: 1.06
```

You can then assign this parameter to a variable like below:

electricity_conversion_factor = parameters(period).electricity_conversion_factor 

Easy!

## Indexed parameters - call a specific value

Of course, you can do something more complex. Let's say you want to have both the electricity conversion and gas conversion factors in the same file. You'd write it something like this. 

```
description: The Electricity Certificate Conversion Factor, applied to the Electricity Savings
  created through any Recognised Energy Savings Activity, and the Gas Certificate Conversion   Factor, applied to the Gas Savings created through any Recognised Energy Savings   Activity,
reference: Equation 1 of the ESS Rule 2020, located in Clause 6.5 of the Energy
  Savings Scheme Rule of 2009. Refers to the Section 130(1) of the Electricity
  Supply Act 1995.
electricity_conversion_factor:
  values:
    2020-01-01:
      value: 1.06
gas_conversion_factor:
  values:
    2020-01-01:
      value: 0.39
```

And then you can refer to specifically the electricity_conversion_factor by doing this: 

electricity_conversion_factor = parameters(period).conversion_factors['electricity_conversion_factor']

What this will do is it'll send the text string in the brackets to the parameters file, try and find something which matches, and then pull that value. 

## Indexed parameters - use an input variable

You can of course feed more than just text strings to a parameters file. For example, see the below example from the CLESF parameters:

```
annual_operating_hours:
  BCA_Class_1a:
    Division_A:
      is_common_area:
        values:
          2020-01-01:
            value: 0
      not_common_area:
        values:
          2020-01-01:
            value: 0
```

You can call the value for a common area, within a Division A business, within a BCA Class 1a type building, with:

```
BCA_Class = 'Class_1a'
building_division = 'Division_A'
common_area = True
is_common_area = where(common_area, 'is_common_area', 'not_common_area')
operating_hours = parameters(period).operating_hours [BCA_Class][building_division][common_area]
```

Parameters also allow for updates to these values with a legislative change. You would do this like the below example, where the conversion factor changes to 1.02 on 1 July 2022:

```
description: The Electricity Certificate Conversion Factor, applied to the Electricity Savings
  created through any Recognised Energy Savings Activity.
reference: Equation 1 of the ESS Rule 2020, located in Clause 6.5 of the Energy
  Savings Scheme Rule of 2009. Refers to the Section 130(1) of the Electricity
  Supply Act 1995.
values:
  2020-01-01:
    value: 1.06
  2022-07-01:
    value: 1.02
```

Now if your simulation has a date after the 2020-01-01 but before 2022-07-01 it'll pull the first value, if it's on or after 2022-07-01 it'll pull the second one.

All of these examples assign the value contained in the parameter file to a variable - after this you can of course modify or manipulate it to your heart's content.

# Tests

Tests in OpenFISCA are written using the YAML markup format. Each variable should be tested with at least one test - and ideally with all of the potential use cases for a variable.

The syntax for a test is below: 

```
- name: 'Text string for the name of the test.'
  period: 2020 # the period of time the test is being conducted for 
  absolute_error_margin: how far out the result can be for the test to still pass, in   terms of a value. i.e. absolute_error_margin of 10 means a result can be 110 or 90,   and still pass if the intended result is 100 [optional]
  relative_error_margin: how far out the result can be for the test to still pass, in   terms of a percentage. i.e. absolute_error_margin of 10 means a result can be 10%   out [optional]
  input: 
    variable_one:
    variable_two:
  output:
    variable_three:
```

So an example of a test, using our coolness_factor variable defined in the logical conditions section: 

``` 
- name: Determine the coolness factor for someone who only plays tennis.
  period: 2021
  input:
    plays_tennis: True
    plays_football: False
  output:
    coolness_factor: 500
```

I'd recommend all of the tests for a particular variable live in their own YAML file, i.e. coolness_factor.yaml, for your ease of use.

## Frequently Asked Questions

You will probably have lots of questions about how to use OpenFISCA - maybe you hit a bug when you try to run a test. Let me know and I'll include them here :)
