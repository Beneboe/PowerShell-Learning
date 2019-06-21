# PowerShell's Equality Operator Behavior

You can define equality for your own classes by creating an `Equals` method.
For example:

```PowerShell
class A {
    [int]$Number

    A ([int]$number) {
        $this.Number = $number
    }

    [boolean] Equals ($other) {
        if ($other -is [A]) {
            Write-Host ("other is A")
            return $this.Number -eq $other.Number
        }
        Write-Host ("other is not A")
        return $false
    }
}
```

Next we will create three instances of the ``A`` class

```PowerShell
$a1 = [A]::new(1)
$a2 = [A]::new(1)
$a3 = [A]::new(2)
```

and compare them:

```PowerShell
$a1.Equals($a1)
# other is A
# True

$a1.Equals($a2)
# other is A
# True

$a1.Equals($a3)
# other is A
# False

$a1.Equals(1)
# other is not A
# False
```

These results are as expected. Interestingly enough, using the `-eq` operator
produces different results.

```PowerShell
$a1 -eq 1
# other is not A
# other is A
# True
```

What's happening here? We can hypothesize that PowerShell does the following:

1. tests if passing the int to the Equals method of A returns true
2. if it returns false, try passing the int to the constructor of A
3. use the new value for comparison

To test this hypothesis we modify the constructor of A

```PowerShell
A ([int]$number) {
    Write-Host "constructor"
    $this.Number = $number
}
```

and assign a new instance to ``$a1`` so that the overwritten class is used.

```PowerShell
$a1 = [A]::new(1)
```

Testing the equality now proves our hypothesis.

```PowerShell
$a1 -eq 1
# other is not A
# constructor
# other is A
# True
```
