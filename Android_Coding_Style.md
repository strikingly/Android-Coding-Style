#Android Coding Style

## Name Conventions

### Class / Interface

First letter capital other small, changing keyword capital and other small. 

```
public class WebsiteManagement(){

}

public interface OnChangeListener(){

}
```

### Field

- Non-public, non-static field names start with m
- Static field names start with s
- Other fields start with a lower case letter
- Public static final fields (constants) are ALL_CAPS_WITH_UNDERSCORES

```
public class MyClass {
    public static final int SOME_CONSTANT = 42;
    public int publicField;
    private static MyClass sSingleton;
    int mPackagePrivate;
    private int mPrivate;
    protected int mProtected;
}
```

### Treat Acronyms as Words

Treat acronyms and abbreviations as words in naming variables, methods, and classes to make names more readable:

Good				| Bad				
------------		| -------------
XmlHttpRequest	| XMLHTTPRequest	
getCustomerId		| getCustomerID
class Html		| class HTML
String url		| String URL
long id			| long ID

## Coding Conventions

### Use TODO Comments

Use TODO comments for code that is temporary, a short-term solution, or good-enough but not perfect. TODOs should include the string TODO in all caps, followed by a colon:

```
// TODO: Change this later
```

### Use Spaces for Indentation

Four (4) space indents for blocks and never tabs. When in doubt, be consistent with the surrounding code.

Eight (8) space indents for line wraps, including function calls and assignments. For example, this is correct:

```
Instrument i =
        someLongExpression(that, wouldNotFit, on, one, line);
```

### Limit Line Length

Each line of text in your code should be at most 100 characters long. While much discussion has surrounded this rule, the decision remains that 100 characters is the maximum with the following exceptions:

- If a comment line contains an example command or a literal URL longer than 100 characters, that line may be longer than 100 characters for ease of cut and paste.

- Import lines can go over the limit because humans rarely see them (this also simplifies tool writing).

### Brace

Braces do not go on their own line; they go on the same line as the code before them:

```
class MyClass {
    int func() {
        if (something) {
            // ...
        } else if (somethingElse) {
            // ...
        } else {
            // ...
        }
    }
}
```

We require braces around the statements for a conditional, even the body just has one statement. No Exception.

```
if (condition) {
    body();
}
```

## Coding Conventions

### Define Fields in Standard Places

Define fields either at the top of the file or immediately before the methods that use them.

### Limit Variable Scope

Keep the scope of local variables to a minimum. By doing so, you increase the readability and maintainability of your code and reduce the likelihood of error. Each variable should be declared in the innermost block that encloses all uses of the variable.

### Write Short Methods

When feasible, keep methods small and focused. We recognize that long methods are sometimes appropriate, so no hard limit is placed on method length. If a method exceeds 50 lines or so, think about whether it can be broken up without harming the structure of the program.

### Fully Qualify Imports

When you want to use class Bar from package foo,there are two possible ways to import it:

`import foo.*;`

Potentially reduces the number of import statements.

`import foo.Bar;`

Makes it obvious what classes are actually used and the code is more readable for maintainers.
Use `import foo.Bar`; for importing all Android code. An **explicit exception** is made for java standard libraries (`java.util.*`, `java.io.*`, etc.) and unit test code (`junit.framework.*`).

### Order Import Statements

The ordering of import statements is:

1.	Android Imports
2. java and javax
3. Imports from third parties

Separated by a blank line between each major grouping.

### Don't Use Finalizers

Finalizers are a way to have a chunk of code executed when an object is garbage collected. While they can be handy for doing cleanup (particularly of external resources), there are no guarantees as to when a finalizer will be called (or even that it will be called at all).

Android doesn't use finalizers. In most cases, you can do what you need from a finalizer with good exception handling. If you absolutely need it, define a close() method (or the like) and document exactly when that method needs to be called (see InputStream for an example). In this case it is appropriate but not required to print a short log message from the finalizer, as long as it is not expected to flood the logs.

### Don't Ignore Exceptions

It can be tempting to write code that completely ignores an exception, such as:

```
void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) { }
}
```

Do not do this. While you may think your code will never encounter this error condition or that it is not important to handle it, ignoring exceptions as above creates mines in your code for someone else to trigger some day. You must handle every Exception in your code in a principled way; the specific handling varies depending on the case.

Acceptable alternatives (in order of preference) are:

- Throw the exception up to the caller of your method.
	
	```
		void setServerPort(String value) throws NumberFormatException {
	    serverPort = Integer.parseInt(value);
	}
	```

- Throw a new exception that's appropriate to your level of abstraction.

	```
	void setServerPort(String value) throws ConfigurationException {
	    try {
	        serverPort = Integer.parseInt(value);
	    } catch (NumberFormatException e) {
	        throw new ConfigurationException("Port " + value + " is not valid.");
	    }
	}
	```

- Handle the error gracefully and substitute an appropriate value in the catch {} block.

	```
	/** Set port. If value is not a valid number, 80 is substituted. */
	
	void setServerPort(String value) {
	    try {
	        serverPort = Integer.parseInt(value);
	    } catch (NumberFormatException e) {
	        serverPort = 80;  // default port for server
	    }
	}
	```

- Catch the Exception and throw a new RuntimeException. This is dangerous, so do it only if you are positive that if this error occurs the appropriate thing to do is crash.

	```
	/** Set port. If value is not a valid number, die. */
	
	void setServerPort(String value) {
	    try {
	        serverPort = Integer.parseInt(value);
	    } catch (NumberFormatException e) {
	        throw new RuntimeException("port " + value " is invalid, ", e);
	    }
	}
	```

- As a last resort, if you are confident that ignoring the exception is appropriate then you may ignore it, but you must also comment why with a good reason:

	```
	/** If value is not a valid number, original port number is used. */
	void setServerPort(String value) {
	    try {
	        serverPort = Integer.parseInt(value);
	    } catch (NumberFormatException e) {
	        // Method is documented to just ignore invalid user input.
	        // serverPort will just be unchanged.
	    }
	}
	
	```

### Don't Catch Generic Exception

It can also be tempting to be lazy when catching exceptions and do something like this:

```
try {
    someComplicatedIOFunction();        // may throw IOException
    someComplicatedParsingFunction();   // may throw ParsingException
    someComplicatedSecurityFunction();  // may throw SecurityException
    // phew, made it all the way
} catch (Exception e) {                 // I'll just catch all exceptions
    handleError();                      // with one generic handler!
}

```

Do not do this. In almost all cases it is inappropriate to catch generic Exception or Throwable (preferably not Throwable because it includes Error exceptions). It is very dangerous because it means that Exceptions you never expected (including RuntimeExceptions like ClassCastException) get caught in application-level error handling. It obscures the failure handling properties of your code, meaning if someone adds a new type of Exception in the code you're calling, the compiler won't help you realize you need to handle the error differently. In most cases you shouldn't be handling different types of exception the same way.

The rare exception to this rule is test code and top-level code where you want to catch all kinds of errors (to prevent them from showing up in a UI, or to keep a batch job running). In these cases you may catch generic Exception (or Throwable) and handle the error appropriately. Think very carefully before doing this, though, and put in comments explaining why it is safe in this place.

Alternatives to catching generic Exception:

- Catch each exception separately as separate catch blocks after a single try. This can be awkward but is still preferable to catching all Exceptions. Beware repeating too much code in the catch blocks.

- Refactor your code to have more fine-grained error handling, with multiple try blocks. Split up the IO from the parsing, handle errors separately in each case.

- Rethrow the exception. Many times you don't need to catch the exception at this level anyway, just let the method throw it.
Remember: exceptions are your friend! When the compiler complains you're not catching an exception, don't scowl. Smile: the compiler just made it easier for you to catch runtime problems in your code.

Remember: exceptions are your friend! When the compiler complains you're not catching an exception, don't scowl. Smile: the compiler just made it easier for you to catch runtime problems in your code.
