```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"strconv"
	"math/rand"
)

func main() {
	// Create variable
	var name string = "JiaoJiao"
	var id int = 112233
	birth_date := "08-04-2000"

	// Print
	fmt.Println(name)
	fmt.Println(id)
	fmt.Println(birth_date)
	fmt.Println(alphabets)
	fmt.Println(fruits)
	fmt.Println(colors)

	// For loop
	for i := 0; i < len(numbers); i++ {
		fmt.Println(numbers[i])
	}

	// Simulating a while loop with for
	i := 0
	for i < 5 {
		fmt.Println(i)
		i++
	}
	
	// Struct & Constructor
	type MinStack struct {
    stack     []int
    sortStack []int
	}
	
	func Constructor() MinStack {
	    return MinStack{
	        stack:     []int{},
	        sortStack: []int{},
	    }
	}
	
	// Switch statement
	color := "red"
	switch color {
	case "red":
		fmt.Println("Color is red")
	case "blue":
		fmt.Println("Color is blue")
	default:
		fmt.Println("Color not recognized")
	}

	// Inner function
	printMessage := func(message string) {
		fmt.Println(message)
	}
	printMessage("Hello from inner function!")

	// Input (simulated for this example)
	fmt.Println("Enter a number:")
	scanner := bufio.NewScanner(os.Stdin)
	scanner.Scan()                           // Waiting for user input
	input, _ := strconv.Atoi(scanner.Text()) // Convert string to int
	fmt.Println("You entered:", input)
}
```

### Array & Slice

```go
	// Create array
	var numbers = [5]int{1, 2, 3, 4, 5}
	alphabets := [3]string{"a", "b", "c"}
	
	// 2d array
	var array [3][2]int

	// Create slice
	fruits := []string{"apple", "banana", "cherry"}
	
	// Create slice twith certain lenth by using make
	ret := make([]string, lens, capa)
	
	// sort
	sort.Ints(numbers)
	sort.Strings(alphabets)
	
	// Delete element
	a = append(a[:index], a[index+1:]...)
	
	// Sort with x[0]
	sort.Slice(data, func(i, j int) bool {
      return data[i][0] < data[j][0]
  })
  
  // Append element at index 0
  newSlice := append([]int{elementToPrepend}, originalSlice...)
  
  // Copy Slice
  original := []int{1, 2, 3, 4, 5}
  newSlice := make([]int, len(original))

	// Random int
	randomIndex := rand.Intn(10)
	
	// Assign one char to another char in string
	runes := []rune(s_1)
	runes[10] = []rune(s_2)[8]
	s_1 := string(runes)
```

### Map

```go
	// Create map
	colors := map[string]string{
		"red":   "#ff0000",
		"green": "#00ff00",
		"blue":  "#0000ff",
	}
	
	// Delete element in map
	delete(colors, "red") 
	
	// Check map element
	_, found := colors["green"]
	if !found {
		// Do something
	}
	
	// Delete key-value
	delete(map, key)
```

### String

```go
	// Split string to slice by " "
	words := strings.Split(str, " ")
	
	// []string to string
	str = strings.Join(strs, " ")
	
	// More same strings
	str = strings.Repeat(" ", 100)
	
	// Check string/rune is alphabet
	unicode.IsLetter(s)
	
	// Check string/rune is digit
	unicode.IsDigit(s)
	
	// Check string/rune is alphabet/digit (Optional)
	func checkAlphanumeric(c byte) bool {
    if (c >= 'A' && c <= 'Z') || (c >= 'a' && c <= 'z') || (c >= '0' && c <= '9') {
        return true
    } 

    return false
	}
	
	// Convert to lower case
	output := strings.ToLower(input)
	
	// Convert to upper case
	output := strings.ToUpper(input)
	
	// Check string contains string
	strings.Contains(source, substring)
	
	// Convert string to int
	num1, err1 := strconv.Atoi(str) 
	
	// Get part of string
	strNew := str[1:5]
	
	// TrimSpace
	trimmedStr := strings.TrimSpace(str)
	
	// Convert string to int
	input, _ := strconv.Atoi(scanner.Text())
	
	// Convert int to string
	str := strconv.Itoa(123)
	fmt.Sprintf("%d", nums[i])
	
	
	str := "Hello, 世界"
	fmt.Println("Using byte:")
	for i := 0; i < len(str); i++ {
	  fmt.Printf("%c ", str[i]) // Byte
	}
	fmt.Println("\nUsing rune:")
	for _, char := range str {
	  fmt.Printf("%c ", char) // Rune
	}
```

### Functions

```go
func reverseSlice(s []interface{}) []interface{} {
	for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
		s[i], s[j] = s[j], s[i]
	}
	return s
}

func Max(x, y int) int {
    if x > y {
        return x
    }
    return y
}
```

```go
go run <go file>
```