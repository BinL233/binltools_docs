### Nearest Smaller to Left (NSL)

```go
func NSL(arr []int) []int {
	n := len(arr)
	result := make([]int, n)

    // Monotonic increasing stack
	stack := []int{} 

	for i := 0; i < n; i++ {
		// Pop elements that are not smaller than the current element
		for len(stack) > 0 && stack[len(stack)-1] >= arr[i] {
			stack = stack[:len(stack)-1]
		}

		// If stack is empty, no smaller element exists to the left
		if len(stack) == 0 {
			result[i] = -1
		} else {
			// The top of the stack is the nearest smaller element
			result[i] = stack[len(stack)-1]
		}

		// Push current element to the stack
		stack = append(stack, arr[i])
	}

	return result
}
```

### Knuth-Morris-Pratt (KMP)

```go
// This function pre-computes the 'Longest Prefix Suffix' table
func computeLPSArray(pattern string) []int {
	m := len(pattern)
	lps := make([]int, m)
	length := 0 // length of the previous longest prefix suffix
	i := 1

	for i < m {
		if pattern[i] == pattern[length] {
			length++
			lps[i] = length
			i++
		} else {
			if length != 0 {
				// Backtrack 'length' to the previous possible prefix
				length = lps[length-1]
			} else {
				lps[i] = 0
				i++
			}
		}
	}
	return lps
}

// KMPSearch finds all occurrences of pattern in text
func KMPSearch(text, pattern string) []int {
	n, m := len(text), len(pattern)
	if m == 0 { return []int{0} }
	
	lps := computeLPSArray(pattern)
	indices := []int{}
	i, j := 0, 0 // i for text, j for pattern

	for i < n {
		if pattern[j] == text[i] {
			i++
			j++
		}

		if j == m {
			// Pattern found! Save the starting index
			indices = append(indices, i-j)
			// Reset j using LPS to find next potential match
			j = lps[j-1]
		} else if i < n && pattern[j] != text[i] {
			// Mismatch after j matches
			if j != 0 {
				j = lps[j-1]
			} else {
				i++
			}
		}
	}
	return indices
}

func main() {
	text := "ABABDABACDABABCABAB"
	pattern := "ABABCABAB"
	matches := KMPSearch(text, pattern)
	
	fmt.Printf("Pattern found at indices: %v\n", matches)
}
```