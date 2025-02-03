# Interpolator (Technical Documentation)

This document contains the technical implementation details of the Interpolator software.

```
Submitted by Team 10 for CS346 Assignment 2.
Team Members:
    Adarsh Gupta (220101003)
    Chouhan Shreyan (220101031)
    D Hayagrivan (220101032)
    Divyansh Chandak (220101039)
    Yash Jain (220101115)
```

# Overall Workflow
The overall technical workflow proceeds as follows:
1. Display Initialisation Prompts: Implemented in the `displayInitPrompt()` function and just displays preset strings.
2. Input File Name from User and Parse the file for data values: Explained in [Data Loading](#data-loading) section.
3. Initialise Interpolator with Points: Explained in [Interpolator Initialisation](#interpolator-initialisation) section.
4. Continously parse user prompt and take appropriate actions: Explained in [User Input Processing](#user-input-processing) section.

The input paired points are represented via tha following structure:
```cpp
struct Point{
	double x, y;

	bool operator<(const Point &other) const{
		return x < other.x;
	}
};
```

Everything is handled via the Interpolation Class which has the following prototype:
```cpp
class Interpolation{
private:
    vector<Point> points; // The inputted array
    
    // Information for Newtons Divided Difference Method, populated during Initialisation step
    vector<double> coefficients;
    vector<double> x_values;
    vector<double> expanded_poly;       // Stores the final polynomial coefficients
    string NewtonPolynomial1;           // Stores intermediate polynomial expression
    string NewtonPolynomial2;           // Stores final polynomial expression

public:
    void processInput(vector<Point>&); // This function is responsible for initialisation of the data structures used in the entire interpolation process
    
    void calculateValue(double x, int method); // This function implements the main user interface to predict the interpolated value for any input via the specified interpolation method

    void NewtonPolynomial(void); // Prints the Newtons Polynomial (both intermediate and final) calculted during initialisation
    void LagrangePolynomial(void); // Prints the Lagrange Polynomial for the interpolation

private:
    // Interpolation functions that are called by calculateValue based on the method
    double piecewiseConstantInterpolation(double x);
    double nearestNeighborInterpolation(double x);
    double linearInterpolation(double x);
    double newtonInterpolation(double x);
    double lagrangeInterpolation(double x);
};
```

## Data Loading
The user provided file name is read into a string called `fileName`. This string is passed as a const char* to the `loadData()` function. The `loadData()` function returns a vector of `Point`'s. This vector contains all the points that were successfully parsed from the user input file, assuming no errors.

```cpp
	string fileName; ...
	getline(cin, fileName);
	vector<Point> points = loadData(fileName.c_str());
```

**Working**: We start by creating a temporary vector of `Point`'s called tempPoints and a file stream object from the user provided filename. At this stage if the file is not open we assume that the user provided filename is either invalid or the user does not have appropriate permission and we raise an error for the same, following which we exit with code 1.

If the file is successfully opened, we parse the file line by line. Each line is read from the file via the `getline()` function. From this line we try to extract the double values of x and y, in case of any error a user warning is raised with the contents of the line. If the values of x and y are successfully parsed from that line, we create a push a `Point` into the temporary vector.

Once all lines in the files are parsed, the `Point`'s are sorted (based on their X values, look at the < operator declaration in the struct). After this 2 security checks are implemented:
1. There exists atleast one point.
2. There does not exist any two points with same X values (checked by comparing adjacent enteries since Points are sorted on X).

The final temporary vector of `Point`'s is returned by the function containig all the values that were successfully read from the file in a sorted order.

## Interpolator Initialisation 
An interpolator object is created and the processInput function is called using the vector of `Point`'s returned by the loadData function.

```cpp
	// Parse File in the interpolator
	interpolater.processInput(points);
```

For interpolation methods 1, 2, 3 and 5 no further processing needs to be done on the input and hence the input `vector<Point>` is directly stored in private variable `points` of the Interpolation class.

However for method 4, i.e. Newton's Divided Difference Method, some preprocessing needs to be done to populate the private variables `coefficients`, `x_values` and `expanded_poly` of the Interpolation class. This function also calculates the string of the Intermediate and Expanded Polynomial form and saves it in the respective string variables to print if `poly` is called by the user.


## User Input Processing
Post initialisation, the user can contiously enter prompts. This part is implemented via an infinite loop:

```cpp
string input; // User Input Line
double X_test; // Inputted X Value
int method; // Inputted Method of interpolation

while true{
    - get user input line from cin

    - check if input is exit then exit
    - check if input is help then print help
    - check if input is poly then call NewtonPolynomial() and LagrangePolynomial() method of the interpolator to print the polynomials

    - if no cases match, extract X_test and method from user input, and print error if unable to do so.
    - call interpolator.calculateValue(X_test, method);
}
```

The `calculateValue()` function does case based matching on the value of the `method` argument. Depending on the value of `method`, it calls the appropriate private interpolation function and the function returns the variable `value`. If method is not in {1, 2, 3, 4, 5} then print error. 

The `value` returned by the specific interpolation function is printed on the console.

The next subsections explain the implementation details of each of the interpolation methods.

### Piecewise Constant Interpolation Implementation

The **Piecewise Constant Interpolation** method, also known as the **nearest left interpolation**, approximates the function as a stepwise constant function. 

So, given a set of sorted points \((x_0, y_0), (x_1, y_1), ..., (x_n, y_n)\), this method finds the interpolated value \( f(x) \) based on the closest point to the left (or the exact match if \( x \) is one of the given data points). Formally, the this can be defined as:

\( f(x) = y_i where x_i <= x < x_{i+1} \)

Edge Cases: If x is outside the given data range:
- For x < x_0, return y_0 (extrapolate using the first data point).
- For x > x_n, return y_n (extrapolate using the last data point).

This behaviour is implemented by using binary search to find the appropriate interval. Since the points are already sorted on their X values during input this allows for a O(log n) implementation.

```cpp
double piecewiseConstantInterpolation(double x){
    - Handle edge cases

    - Binary Search to find correct interval
    low = 0, high = points.size() - 1;
    while (low <= high){
        - calculate mid = middle index

        - if x value of mid Point <= x and x value of mid + 1 > x
          then we have found the appropriate interval, so return y value of mid Point

        - otherwise if x value of mid point > x
                    then set high = mid - 1
                    else set low = mid + 1
    }
}
```

### Nearest Neighbor Interpolation Implementation
**- TBD (Shreyan)**

### Linear Interpolation Implementation

**Linear Interpolation** approximates the function by connecting consecutive data points with straight lines. For a given x, it identifies the two surrounding points and computes f(x) using the linear equation between them. This method ensures continuity between adjacent intervals but does not guarantee smoothness.

**Formula**  
For x in the interval \( x_i, x_{i+1} \), the interpolated value is:  

f(x) = y_i + (y_{i+1} - y_i)/(x_{i+1} - x_i) *(x - x_i)

**Edge Cases**  
- If \( x < x_0 \), return y_0  (extrapolate using the first point).  
- If \( x >= x_n \), return  y_n  (extrapolate using the last point).  

**Implementation Steps**  
1. **Sort Points**: Ensure data points are sorted by x -values.  
2. **Interval Search**: Use binary search (`lower_bound`) to find the first point \( x_r \geq x \). The left point is the previous index.  
3. **Edge Handling**: If  x  is outside the data range, return the nearest endpoint's y -value.  
4. **Linear Calculation**: Compute the interpolated  y -value using the linear formula between \( (x_l, y_l) \) and \( (x_r, y_l) \).  

**Code**  
```cpp
double linearInterpolation(double x) {
    int n = points.size();
    sort(points.begin(), points.end()); // Ensure sorted order

    Point p = {x, 0.0};
    auto it = lower_bound(points.begin(), points.end(), p);
    
    // Edge cases: x outside data range
    if (it == points.end()) return points[n - 1].y;
    if (it == points.begin()) return points[0].y;

    int r = it - points.begin();
    int l = r - 1;

    // Linear formula application
    double slope = (points[r].y - points[l].y) / (points[r].x - points[l].x);
    return points[l].y + slope * (x - points[l].x);
}
```

This method is efficient for dense datasets and provides piecewise linear approximations.

### Newton’s Divided Difference Interpolation Implementation

Newton’s **Divided Difference Interpolation** constructs an interpolating polynomial using recursively computed divided differences. Given a sorted set of points:

\[
(x_0, y_0), (x_1, y_1), ... , (x_n, y_n)
\]

the interpolating polynomial is constructed in the form:

\[
P(x) = f[x_0] + f[x_0, x_1](x - x_0) + f[x_0, x_1, x_2](x - x_0)(x - x_1) + ...
\]

where **\( f[x_0, x_1, ..., x_k] \)** are the **divided differences**, computed using:

\[
f[x_i, x_{i+1}, ..., x_{i+k}] = {f[x_{i+1}, ..., x_{i+k}] - f[x_i, ..., x_{i+k-1}]} / {x_{i+k} - x_i}
\]

Before interpolation, we need to:
1. **Extract the input data** into `x_values` and `y_values`.
2. **Compute the divided difference table** to find polynomial coefficients.
3. **Store intermediate and expanded polynomial forms**.

This preprocessing is done in the `processInput()` function.

#### Computing the Divided Difference Table
```cpp
// Initialize first column with given y values
for (int i = 0; i < n; i++) {
    divided_diff[i][0] = y[i];
}

// Compute divided differences iteratively
for (int j = 1; j < n; j++) {
    for (int i = 0; i < n - j; i++) {
        divided_diff[i][j] = (divided_diff[i + 1][j - 1] - divided_diff[i][j - 1]) / (x[i + j] - x[i]);
    }
}
```
**Explanation:**
- The first column is simply the function values **\( y_i \)**.
- Each subsequent column is computed recursively using divided differences.

#### Extracting Newton's Coefficients
```cpp
// Store coefficients of Newton's polynomial
coefficients.clear();
for (int i = 0; i < n; i++) {
    coefficients.push_back(divided_diff[0][i]);
}
```
These coefficients will later be used to evaluate the interpolating polynomial.

#### Constructing the Polynomial Representation

##### **Intermediate Polynomial Form** (Newton’s Recursive Form)
```cpp
ostringstream oss;
oss << fixed << setprecision(2);
oss << "P(x) = " << divided_diff[0][0];  // Start with first term
for (int i = 1; i < n; i++) {
    oss << " + (" << divided_diff[0][i] << ")";
    for (int j = 0; j < i; j++) {
        oss << "(x - " << x[j] << ")";
    }
}
oss << endl;
NewtonPolynomial1 = oss.str();
```
This constructs the recursive polynomial form, which looks like:

\[
P(x) = a_0 + a_1 (x - x_0) + a_2 (x - x_0)(x - x_1) + ...
\]

##### **Expanded Polynomial Form**
To convert the recursive form into a **fully expanded polynomial**, we perform polynomial multiplication:

```cpp
// Construct final polynomial in expanded form
vector<double> poly(1, 1.0); // Start with constant term 1
expanded_poly.assign(n, 0);

for (int i = 0; i < n; i++) {
    for (int j = 0; j < poly.size(); j++) {
        expanded_poly[j] += coefficients[i] * poly[j]; // Multiply coefficient
    }
    if (i < n - 1) {
        poly = multiplyPoly(poly, x[i]); // Multiply by (x - x_i)
    }
}
```
This ensures that:

\[
P(x) = c_0 + c_1 x + c_2 x^2 + ...
\]

where `expanded_poly` stores the coefficients **\( c_i \)**.

Finally, we format this into a readable string:
```cpp
stringstream ss;
ss << fixed << setprecision(2);

bool first_term = true;
for (int i = expanded_poly.size() - 1; i >= 0; i--) {
    if (fabs(expanded_poly[i]) > 1e-9) { // Ignore near-zero coefficients
        if (!first_term) {
            ss << (expanded_poly[i] > 0 ? " + " : " - ");
        }
        first_term = false;
        ss << fabs(expanded_poly[i]);
        if (i > 0) ss << "x";
        if (i > 1) ss << "^" << i;
    }
}
ss << endl;

NewtonPolynomial2 = ss.str();
```

#### **Polynomial Evaluation**
Once the coefficients are precomputed, **evaluating \( P(x) \) for any given \( x \)** can be done in **O(n) time** using:

```cpp
double newtonInterpolation(double x)
{
    int n = coefficients.size();
    double result = coefficients[0];  // Start with first term

    double term = 1;
    for (int i = 1; i < n; i++) {
        term *= (x - x_values[i - 1]);  // Compute (x - x0)(x - x1)...(x - xi-1)
        result += coefficients[i] * term;
    }
    return result;
}
```
This efficiently evaluates the polynomial using nested multiplication (Horner's method).

In summary, the divided difference table is precomputed in O(n²) time, following which the intermediate and expanded polynomials are calculated and stored as string. These strings can just be printed whenever the user calls the `poly` command. Evaluation is done with the help of Horner's method in O(n) time.


### Lagrange Interpolation Implementation

**Lagrange Interpolation** is a method used to construct a polynomial that exactly fits a given set of data points. The polynomial is constructed by using a weighted sum of Lagrange basis polynomials. Each basis polynomial is designed such that it is 1 at one specific data point and 0 at all other data points.

The function computes the value of the interpolating polynomial for a given \( X \) by evaluating the sum of all the Lagrange basis polynomials, each weighted by the corresponding \( y \)-value.

#### Steps:

1. **Initialize** the result to 0.0.
2. For each data point \( (x_i, y_i) \):
   - Compute the Lagrange basis polynomial \( L_i(x) \) by iterating over all other data points \( j \neq i \).
   - Multiply the result by the corresponding \( y_i \).
3. Return the sum of all the weighted terms as the interpolated value at \( X \).

#### Code Implementation:

```cpp
double lagrangeInterpolation(double X)
{
    // Initialize the result to 0.0
    double result = 0.0;
    int n = points.size();

    // Iterate over each point in the dataset
    for (int i = 0; i < n; i++)
    {
        double term = points[i].y; // Start with the y-value at point i

        // Compute the Lagrange basis polynomial L_i(x)
        for (int j = 0; j < n; j++)
        {
            if (j != i)
            {
                // Ensure there are no duplicate x-values
                if (points[i].x == points[j].x)
                {
                    cerr << "[Error] Duplicate x-values detected, which will result in division by zero." << endl;
                    exit(1);
                }

                // Multiply by the fraction for L_i(x)
                term *= (X - points[j].x) / (points[i].x - points[j].x);
            }
        }

        // Add the weighted term to the result
        result += term;
    }

    return result;
}
```

#### Explanation:

1. **Outer loop**: Iterates over all points in the dataset to compute each Lagrange basis polynomial \( L_i(x) \).
2. **Inner loop**: Computes the product for each Lagrange basis polynomial \( L_i(x) \) by iterating over all other points \( j \neq i \).
3. The result is accumulated by adding each term, which is weighted by the corresponding \( y_i \).
4. The final result is returned as the interpolated value at the given \( X \).

#### Polynomial Evaluation:

1. **Purpose**:
   1. Constructs interpolating polynomial passing through given points
   2. Outputs formatted polynomial string

```cpp
void LagrangePolynomial(void)
	{
        //1. Initialisation
        //2. Calculate Coefficients
        //3. Format and print ouytput
    }

```

2. **Algorithm Steps**:

A. *Initialize*:

- Check for empty points
- Setup coefficient vector

```cpp
    int n = points.size();
    if (n == 0)
    {
        cout << 0 << endl; // Return zero polynomial for empty input
    }

    vector<double> finalCoeff(n, 0.0);
```

B. *Calculate Coefficients*:

- A[Start] --> B[For each point i]
- B --> C[Initialize basis term]
- C --> D[For each point j≠i]
- D --> E[Multiply by (x-xⱼ)/(xᵢ-xⱼ)]
- E --> F[Multiply by yᵢ]
- F --> G[Add to final coefficients]

```cpp
    // Compute the coefficients of the polynomial
		for (int i = 0; i < n; ++i)
		-
			vector<double> term(1, 1.0); // Start with a constant 1 for L_i(x)

			for (int j = 0; j < n; ++j)
			{
				if (j != i)
				{
					// Multiply the term by (x - points[j].x)
					vector<double> temp(term.size() + 1, 0.0);

					for (int k = 0; k < term.size(); ++k)
					{
						temp[k] -= term[k] * points[j].x; // Multiply by (-points[j].x)
						temp[k + 1] += term[k];			  // Multiply by x
					}

					term = temp;

					// Divide by (points[i].x - points[j].x)
					double denominator = points[i].x - points[j].x;
					for (double &coeff : term)
					{
						coeff /= denominator;
					}
				}
			}

			// Multiply by y-value and add to final coefficients
			for (int k = 0; k < term.size(); ++k)
			{
				finalCoeff[k] += term[k] * points[i].y;
			}


```

C.  *Format Output and Print it*:

- Highest degree terms first
- Include proper signs
- Add exponents

```cpp
    // Format the polynomial string
		stringstream ss;
		bool first = true;

		for (int i = finalCoeff.size() - 1; i >= 0; --i)
		{
			if (abs(finalCoeff[i]) < 1e-10)
				continue; // Skip near-zero coefficients

			if (!first && finalCoeff[i] > 0)
				ss << "+";
			if (abs(finalCoeff[i] - 1) > 1e-10 || i == 0)
				ss << fixed << setprecision(4) << finalCoeff[i];

			if (i > 0)
			{
				ss << "x";
				if (i > 1)
					ss << "^" << i;
			}

			first = false;
		}

		string result = ss.str();
		if (result.empty())
		{
			cout << 0 << endl;
		}
		else
		{
			cout << result << endl;
		}

```

