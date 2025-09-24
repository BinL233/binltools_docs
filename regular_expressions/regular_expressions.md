```C
// ...abc...
"abc"

// Any char
"."

// Only match "." because inside "[]", "." losses its functionality
"[.]"

// Set: Any char in set
"[aeiou]"

// Any upper letter
"[A-Z]"

// Any char except digits “0” through “9”, period, and plus sign
"[^0-9.+]"

// "X" or ""
"X?"

// "" or more "XX..."
"X*"

// "X" or more "XX..."
"X+"

// "X" or "Y"
"X|Y"

// "abc...."
"^abc"

// "....abc"
"abc$"

// Any string with "X"
".*X.*"

// Extended
"X{m,n}" // At least m "X" and at most n "X"
"X{M,}"  // At least m "X"
"X{,n}"  // At most n "X"

// Matches the group 1 and back reference
// "([a-z]{2,3})^\1" match "abc^abc"
"\1"
"\2" // Group 2
```