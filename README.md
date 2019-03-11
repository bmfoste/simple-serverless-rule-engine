# simple-serverless-rule-engine
A __lightweight__ yet __powerful__ rule engine that allows declarative specification of business rules and **saves tons of repeated development work**.

## Key Features
1. Ability to __declaratively__ author both Scoring and Decision Rules.
2. Ability to __version control__ rule declarations thus enabling auditing of rule changes over a period of time.
3. Ability to author _chained rules_. Evaluation of one rule can refer to the result of another rule, thus enabling 
modular, hierarchical rules. 
4. The rule engine is state-less and server less - perfect for hosting it as an independent micro-service.
5. Written in Python 3.6 

## Installation Instructions
1. Clone or download the project into your local repository.
2. Create a virtual environment with Python 3.6 or above and activate the same.
```python
virtualenv .env --python=Pyhton3.6
source .env/bin/activate
```
3. To deploy this as a FaaS through Zappa, use [Zappa](https://www.zappa.io/), a framework for Serverless Python Web Services - Powered by AWS Lambda and API Gateway
```python
zappa deploy {{your stage name}}
```
4. To deploy this in a Kubernetes container, use [Fission](https://fission.io/), a framework for serverless functions on Kubernetes.
```python

```


# Why Rule Engine?
Decision making has always been at the heart of any business. In certain industries (such as Lending), some of the decisions made are so dynamic & at a flux that programming these decisions by hand is counter-productive.

Take the example of the decision of giving someone a loan. It primarily involves ascertaining to fundamental factors:
- Ability to repay the loan.
- Intent to repay the loan.

When you start assessing a borrower based on above, you typically get all facts required to make a decision (such as Bureau score, bank statements etc.) and you will pass these facts through a decision matrix to arrive at 
- A composite score on a scale that gives an indication of _whether the borrower will repay the loan_ (**intent**)
- A recommendation of _how much loan_ should be given to the borrower. (**ability**)

The aforementioned decisions involve evaluation of multiple parameters. You simply cannot write a program to solve such complex scoring or decision problems:
 - The evaluations and/or scores will _always_ change over a period of time to adjust to business needs 
- The _rules_ will also change based on the nature of the business product.

**The simple-serverless-rule-engine solves such dynamic decision making problems by abstracting the scoring or decision making into an engine and providing a standard rule template (JSON) to author the decisions.** 
By doing so, we can conveniently treat the rule engine as a service and just by passing all the facts (a.k.a inputs), we get the the corresponding decisions or scores (output)!

### Benefits
- Declarative authoring of rules. This can be done by a business analyst, rather than a developer. The developer just focuses on extracting the facts that are required to be passed into the engine.
- Versioning of rules to enable the auditing of the rule changes over a period of time.

# Concepts

The simple-serverless-rule-engine is composed of two parts:

- A Rule template which enables one to declaratively specify a rule, which can either be a Decision (or) a Score. The rule template is uniquely identified by a name.
- A parser engine which when passed with a rule name & facts, parses the rule template against the facts given and provides an output. The output can either be a score (numeric) or a decision (anything).

The simple-serverless-rule-engine allows the rules to be _“chained”_. I.e. you can build a small portion of the score as an independent rule and _“use”_ this rule in another rule. 

**At the heart of simple-serverless-rule-engine is the rule declaration language.** 

A rule can either be a Decision or a Score.

## Score rule:
- A Score rule is composed of one or many rule sets. 
- Each rule-set computes a sub-score and is applied a weight. 
- The total score then would be the sum of all the individual scores of all the rule sets belonging to a rule.
- A rule set is composed of one or many rule rows. 
- You can ‘roughly’ think of each Rule Row as a Conditional evaluation of the facts (a.k.a antecedent) & a score based on these conditions (a.k.a consequent).
 
[Score Rule Concept](https://photos.app.goo.gl/584J4aZCsLEHHT6aA "Score Rule")

## Decision rule:
- A Decision rule is always composed of only one rule set.
- A rule set is composed of one or many rule rows. 
- You can ‘roughly’ think of each Rule Row as a Conditional evaluation of the facts (a.k.a antecedent) & a score based on these conditions (a.k.a consequent).
- A decision rule always arrives at a single decision at the end of parsing.
- The decision can be anything (a numeric, a string such as YES/NO or even a JSON)

[Decision Rule Concept](https://photos.app.goo.gl/RxisVyVJ1tTqBCmP9 "Decision Rule")

## Antecedent and Consequent

- An antecedent at the core is an evaluator. It evaluates one or many facts through an operator.
- For evaluating numeric facts, a numeric operator is used. It can be one of (<=, <, >, >=, ==, <>, between, is_none)
- For evaluating string facts, a string operator is used. It can be one of (in_list, contains, is_none, equals)
- You can mix evaluation of more than one fact & combine the result with an “and” or “or” condition.

You can perform complex evaluations of multiple facts recursively in the antecedent.

Example: Let us say that you want to set a score based on the following condition:

(The applicant age is > 35 AND applicant ownership is Owned) OR business ownership is Owned.
OR
(The applicant age is <= 35 AND applicant ownership is Rented) OR business ownership is Rented.

This would be modeled as {TBD: Showcase the Rule template for this scenario}

The system allows a total recursion depth of 5 to allow complex evaluations.

Pro tip: You can use the result of a rule as a token! This way you can build simple modular rules & combine them to get to a bigger rule. For example, you can use a particular decision to feed into a bigger decision.

The consequent tells the rule engine what to do when the antecedent is evaluated as True. In the case of a score rule, you must specify a score value (numeric).

In the case of a Decision rule, you must specify a decision. The decision can be a simple value (0 or 1, yes or no etc.) (or) it can be a complex JSON in itself!

# Examples