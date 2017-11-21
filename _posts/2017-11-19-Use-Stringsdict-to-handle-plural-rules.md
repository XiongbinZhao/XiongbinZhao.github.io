---
layout:     post                    
title:      Use stringsdict to handle plural rules
subtitle:   
date:       2017-11-19
author:     Xiongbin Zhao
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - iOS
---

# Use Stringsdict file to handle plural in your app


When developing app, it is not uncommon that we need to handle plural strings because language have their plural rules. So, we will have this kind of logic all over our app:

```
if (number == 1) {
	string = @"item";
    } else {
    string = @"items";
}
```
This code is simple and straighforward. So, maybe it is not a big deal to have this code all over our app. But things get complicated when we are trying to handle other languages whose plural rule would be much more complicated that English.

## Why we need Stringsdict
In English, the plural rule is simple. There is only two cases, `one` and `other`. That is why there are only two branches in the example code above.

However, for other languages, like Russian, the plural rules is much more complicated.

```
Numbers ending in: 1
один рубль - one rouble

Numbers ending in: 2,3,4
три рубля - three roubles

Numbers ending in: 5,6,7,8,9,0
пять рублей - five roubles

```
You will need more code to support this kind of plural rules. And it make inernationaliztion even harder because you need to support different languages.

## What is a Stringsdict


[Stringsdict](https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPInternational/StringsdictFileFormat/StringsdictFileFormat.html#//apple_ref/doc/uid/10000171i-CH16-SW1) is a property list used to define language plural rules. Here is is what a stringsdict looks like.

```
<dict>
	<key>kExampleKey</key>
	<dict>
		<key>NSStringLocalizedFormatKey</key>
		<string>%#@item@</string>
		<key>item</key>
		<dict>
			<key>NSStringFormatSpecTypeKey</key>
			<string>NSStringPluralRuleType</string>
			<key>NSStringFormatValueTypeKey</key>
			<string>d</string>
			<key>one</key>
			<string>%d item</string>
			<key>other</key>
			<string>%d items</string>
		</dict>
	</dict>
</dict>

```

Stringsdict is a key value pair for the language plural. In our case, our key is `kExampleKey`, whose value is a dictionary of our plural rule.

### NSStringLocalizedFormatKey
NSStringLocalizedFormatKey is used to define our variable between **%#@** and **@**. In our case we set this field to **%#@item@**, which means our variable is **item**. We can define multiple variable in this field, as a result, we need to create plural rules for each variable we've defined. So, we can define a FormatString with two variables like this:

```
<key>NSStringLocalizedFormatKey</key>
<string>We will have %#@num_of_item@ for %#@num_of_people@</string>
```

Therefore, we will have two variables, num\_of\_item and num\_of\_people.

For each variable we defined, we need to create our plural rule.

### Plural Rule Property


##### NSStringFormatSpecTypeKey
NSStringFormatSpecTypeKey can only has one value: NSStringPluralRuleType.


##### NSStringFormatValueTypeKey

NSStringFormatValueTypeKey is used to define the [string format specifier](https://developer.apple.com/library/content/documentation/CoreFoundation/Conceptual/CFStrings/formatSpecifiers.html#//apple_ref/doc/uid/TP40004265) of our variable, like **d** for integer, **f** for float

**zero**:
The format string to use for number 0

**one**:
The format string to use for additional language-dependent categories.

**two**
The format string to use for additional language-dependent categories.

**few,many**
Format strings to use for additional language-dependent categories.

**other (Required)**
The format string to use for all number not converted by the other categories.

\*You don't need to use the number defined for `NSStringFormatValueTypeKey`. For example, for case `one` you can use `one person`

\*[CLDR Plural Rules](http://www.unicode.org/cldr/charts/latest/supplemental/language_plural_rules.html)

## How to use a stringsdict

To use a stringsdict, you need to create a stringsdict file under your xcode project.

For example, we can create a `Localizable.stringsdict` in our project like this:

```
<dict>
	<key>kExampleKey</key>
	<dict>
		<key>NSStringLocalizedFormatKey</key>
		<string>We will have %#@num_of_item@ for %#@num_of_people@</string>
		<key>num_of_item</key>
		<dict>
			<key>NSStringFormatSpecTypeKey</key>
			<string>NSStringPluralRuleType</string>
			<key>NSStringFormatValueTypeKey</key>
			<string>d</string>
			<key>one</key>
			<string>%d MacBook</string>
			<key>other</key>
			<string>more than one MacBooks</string>
		</dict>
		<key>num_of_people</key>
		<dict>
			<key>NSStringFormatSpecTypeKey</key>
			<string>NSStringPluralRuleType</string>
			<key>NSStringFormatValueTypeKey</key>
			<string>d</string>
			<key>one</key>
			<string>each person</string>
			<key>other</key>
			<string>every %d people</string>
		</dict>
	</dict>
</dict>

```
For this case, we create two different variables num_of_item and num_of_people. When we use this stringsdict, when num\_of\_item is 1, we will get `1 MacBook`, for the other case, we will get `every %d people`

Now, in our code, we can create a uilabel to use this string:

```
UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(0, 100, 300, 200)];
label.text = [NSString stringWithFormat:NSLocalizedString(@"example", nil), 1, 3];
```
Our label will show: **We will have 1 MacBook for every 3 people**

```
UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(0, 100, 300, 200)];
label.text = [NSString stringWithFormat:NSLocalizedString(@"example", nil), 2, 1];
```
Our label will show: **We will have more than one MacBook for each person**

In conclude, the string we defined for each category will replace our variable based on the number we pass in for the string and we don't have to include our variable number in our string.


## About NSLocalizedString method
In addition, there are four method for getting Localized String, which have different default value:

```
NSLocalizedString(key, comment)
// Get string for "key" from Localizable.strings or Localizable.stringsdict table in Main bundle

NSLocalizedStringFromTable(key, tbl, comment)
// Get string for "key" from specified Table: "tbl", in Main bundle

NSLocalizedStringFromTableInBundle(key, tbl, bundle, comment)
// Get string for "key" from specified Table: "tbl", in "bundle"

NSLocalizedStringWithDefaultValue(key, tbl, bundle, val, comment)
// Get string for "key" from specified Table: "tbl", in "bundle". If value not found, return "val" as default value

//*comment is used to generate comment when creating strings file using genstring command

```
