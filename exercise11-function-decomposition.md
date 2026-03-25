# Exercise 11: Function Decomposition Challenge

## Function Selected
Function 1: validateUserData() — JavaScript
Data Validation Function with Nested Conditionals

---

## Original Function

The original validateUserData() function is a single function of approximately
130 lines that handles all of the following responsibilities in one place:

- Checking required fields for registration vs profile update
- Validating username format and uniqueness
- Validating password strength and confirmation match
- Validating email format and uniqueness
- Validating date of birth (format, age range)
- Validating address structure and country-specific postal codes
- Validating phone number format
- Running custom user-provided validations

This violates the Single Responsibility Principle — the function does
8 different jobs and is extremely difficult to test, read, or modify.

---

## Step 1: Identifying Distinct Responsibilities

### AI Prompt Used (Function Responsibility Analysis)
```
I have a complex function that I'd like to refactor by breaking it into
smaller functions:

[pasted the full validateUserData function]

Please:
1. Identify the distinct responsibilities/concerns in this function
2. Suggest a decomposition strategy with smaller functions
3. Show which parts of the original code would move to each new function
4. Provide a refactored version with the main function delegating
   to the smaller functions
```

### Responsibilities Identified

| Responsibility | Lines in Original | New Function Name |
|----------------|-------------------|-------------------|
| Check required fields based on operation type | Lines 8-17 | validateRequiredFields() |
| Validate username rules and uniqueness | Lines 19-32 | validateUsername() |
| Validate password strength and confirmation | Lines 34-53 | validatePassword() |
| Validate email format and uniqueness | Lines 55-70 | validateEmail() |
| Validate date of birth range and format | Lines 72-92 | validateDateOfBirth() |
| Validate address structure and postal codes | Lines 94-126 | validateAddress() |
| Validate phone number format | Lines 128-133 | validatePhone() |
| Run custom validations | Lines 135-145 | runCustomValidations() |
| Coordinate all validations and return errors | Main function | validateUserData() (orchestrator) |

---

## Step 2: Decomposition Plan

The main function becomes a thin **orchestrator** — it calls each validator
and collects errors. Each validator focuses on one thing only and returns
an array of error strings (empty if valid).
```
validateUserData(userData, options)
  ↓
  ├── validateRequiredFields(userData, isRegistration)
  ├── validateUsername(userData, options.checkExisting)    [only if registration]
  ├── validatePassword(userData)                           [only if registration]
  ├── validateEmail(userData, options.checkExisting, isRegistration)
  ├── validateDateOfBirth(userData)
  ├── validateAddress(userData)
  ├── validatePhone(userData)
  └── runCustomValidations(userData, options.customValidations)
```

---

## Step 3: Refactored Implementation
```javascript
/**
 * Validates user input for registration or profile updates.
 * Orchestrates all validation checks and returns combined errors.
 *
 * @param {Object} userData - The user data to validate
 * @param {Object} options - Validation options
 * @param {boolean} options.isRegistration - True for new user registration
 * @param {Object} options.checkExisting - Object with usernameExists/emailExists methods
 * @param {Array} options.customValidations - Array of custom validation rules
 * @returns {string[]} Array of validation error messages (empty if valid)
 */
function validateUserData(userData, options = {}) {
  const isRegistration = options.isRegistration || false;
  let errors = [];

  // Each validator returns an array of error strings
  errors = errors.concat(validateRequiredFields(userData, isRegistration));

  if (isRegistration) {
    errors = errors.concat(validateUsername(userData, options.checkExisting));
    errors = errors.concat(validatePassword(userData));
  }

  errors = errors.concat(validateEmail(userData, options.checkExisting, isRegistration));
  errors = errors.concat(validateDateOfBirth(userData));
  errors = errors.concat(validateAddress(userData));
  errors = errors.concat(validatePhone(userData));
  errors = errors.concat(runCustomValidations(userData, options.customValidations));

  return errors;
}

// ─────────────────────────────────────────────
// EXTRACTED HELPER FUNCTIONS
// ─────────────────────────────────────────────

/**
 * Checks that all fields required for the operation type are present.
 */
function validateRequiredFields(userData, isRegistration) {
  const errors = [];
  const requiredFields = isRegistration
    ? ['username', 'email', 'password', 'confirmPassword']
    : ['firstName', 'lastName', 'dateOfBirth', 'address'];

  for (const field of requiredFields) {
    if (!userData[field] || userData[field].toString().trim() === '') {
      errors.push(`${field} is required for ${isRegistration ? 'registration' : 'profile update'}`);
    }
  }

  return errors;
}

/**
 * Validates username length, character rules, and uniqueness.
 */
function validateUsername(userData, checkExisting) {
  const errors = [];
  const { username } = userData;

  if (!username) return errors; // Missing field handled by validateRequiredFields

  if (username.length < 3) {
    errors.push('Username must be at least 3 characters long');
  } else if (username.length > 20) {
    errors.push('Username must be at most 20 characters long');
  } else if (!/^[a-zA-Z0-9_]+$/.test(username)) {
    errors.push('Username can only contain letters, numbers, and underscores');
  } else if (checkExisting && checkExisting.usernameExists(username)) {
    errors.push('Username is already taken');
  }

  return errors;
}

/**
 * Validates password strength requirements and confirmation match.
 */
function validatePassword(userData) {
  const errors = [];
  const { password, confirmPassword } = userData;

  if (!password) return errors; // Missing field handled elsewhere

  const strengthRules = [
    { test: pwd => pwd.length >= 8,         message: 'Password must be at least 8 characters long' },
    { test: pwd => /[A-Z]/.test(pwd),        message: 'Password must contain at least one uppercase letter' },
    { test: pwd => /[a-z]/.test(pwd),        message: 'Password must contain at least one lowercase letter' },
    { test: pwd => /[0-9]/.test(pwd),        message: 'Password must contain at least one number' },
    { test: pwd => /[^A-Za-z0-9]/.test(pwd), message: 'Password must contain at least one special character' }
  ];

  for (const rule of strengthRules) {
    if (!rule.test(password)) {
      errors.push(rule.message);
      break; // Report one password error at a time
    }
  }

  if (errors.length === 0 && confirmPassword !== password) {
    errors.push('Password and confirmation do not match');
  }

  return errors;
}

/**
 * Validates email format and uniqueness.
 */
function validateEmail(userData, checkExisting, isRegistration) {
  const errors = [];
  const email = userData.email;

  if (email === undefined) return errors;

  if (email.trim() === '') {
    if (isRegistration) errors.push('Email is required');
    return errors;
  }

  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(email)) {
    errors.push('Email format is invalid');
  } else if (checkExisting && checkExisting.emailExists(email)) {
    errors.push('Email is already registered');
  }

  return errors;
}

/**
 * Validates date of birth format and age range (must be 13-120 years old).
 */
function validateDateOfBirth(userData) {
  const errors = [];
  const { dateOfBirth } = userData;

  if (!dateOfBirth || dateOfBirth === '') return errors;

  const dobDate = new Date(dateOfBirth);
  if (isNaN(dobDate.getTime())) {
    errors.push('Date of birth is not a valid date');
    return errors;
  }

  const now = new Date();
  const minAgeDate = new Date(now.getFullYear() - 13, now.getMonth(), now.getDate());
  const maxAgeDate = new Date(now.getFullYear() - 120, now.getMonth(), now.getDate());

  if (dobDate > now)         errors.push('Date of birth cannot be in the future');
  else if (dobDate > minAgeDate) errors.push('You must be at least 13 years old');
  else if (dobDate < maxAgeDate) errors.push('Invalid date of birth (age > 120 years)');

  return errors;
}

/**
 * Validates address structure, required fields, and country-specific postal codes.
 */
function validateAddress(userData) {
  const errors = [];
  const { address } = userData;

  if (!address || address === '') return errors;

  if (typeof address !== 'object') {
    errors.push('Address must be an object with required fields');
    return errors;
  }

  const requiredAddressFields = ['street', 'city', 'zip', 'country'];
  for (const field of requiredAddressFields) {
    if (!address[field] || address[field].trim() === '') {
      errors.push(`Address ${field} is required`);
    }
  }

  // Validate postal code format if both zip and country are provided
  if (address.zip && address.country) {
    const postalCodeError = validatePostalCodeForCountry(address.zip, address.country);
    if (postalCodeError) errors.push(postalCodeError);
  }

  return errors;
}

/**
 * Returns an error message if the postal code format is invalid for the country,
 * or null if valid or country is not specifically validated.
 */
function validatePostalCodeForCountry(zip, country) {
  const formats = {
    US: { regex: /^\d{5}(-\d{4})?$/,                  message: 'Invalid US ZIP code format' },
    CA: { regex: /^[A-Za-z]\d[A-Za-z] \d[A-Za-z]\d$/, message: 'Invalid Canadian postal code format' },
    UK: { regex: /^[A-Z]{1,2}\d[A-Z\d]? \d[A-Z]{2}$/, message: 'Invalid UK postal code format' }
  };

  const format = formats[country];
  if (format && !format.regex.test(zip)) return format.message;
  return null;
}

/**
 * Validates phone number format using a basic international pattern.
 */
function validatePhone(userData) {
  const errors = [];
  const { phone } = userData;

  if (!phone || phone === '') return errors;

  if (!/^\+?[\d\s\-()]{10,15}$/.test(phone)) {
    errors.push('Phone number format is invalid');
  }

  return errors;
}

/**
 * Runs any custom validation rules provided by the caller.
 */
function runCustomValidations(userData, customValidations) {
  if (!customValidations) return [];

  const errors = [];

  for (const validation of customValidations) {
    const { field, validator, message } = validation;
    if (userData[field] !== undefined && !validator(userData[field], userData)) {
      errors.push(message || `Invalid value for ${field}`);
    }
  }

  return errors;
}
```

---

## Step 4: Verification That Behavior Is Preserved

### Tests Written to Confirm Correct Refactoring
```javascript
describe('validateUserData — refactoring verification', () => {

  test('registration with all valid data returns no errors', () => {
    const userData = {
      username: 'john_doe',
      email: 'john@example.com',
      password: 'SecurePass1!',
      confirmPassword: 'SecurePass1!'
    };
    const options = { isRegistration: true };
    expect(validateUserData(userData, options)).toEqual([]);
  });

  test('registration with missing username returns error', () => {
    const userData = {
      email: 'john@example.com',
      password: 'SecurePass1!',
      confirmPassword: 'SecurePass1!'
    };
    const options = { isRegistration: true };
    const errors = validateUserData(userData, options);
    expect(errors).toContain('username is required for registration');
  });

  test('short password returns specific error', () => {
    const userData = {
      username: 'john_doe',
      email: 'john@example.com',
      password: 'short',
      confirmPassword: 'short'
    };
    const options = { isRegistration: true };
    const errors = validateUserData(userData, options);
    expect(errors).toContain('Password must be at least 8 characters long');
  });

  test('password mismatch returns specific error', () => {
    const userData = {
      username: 'john_doe',
      email: 'john@example.com',
      password: 'SecurePass1!',
      confirmPassword: 'DifferentPass1!'
    };
    const options = { isRegistration: true };
    const errors = validateUserData(userData, options);
    expect(errors).toContain('Password and confirmation do not match');
  });

  test('invalid email format returns specific error', () => {
    const userData = {
      username: 'john_doe',
      email: 'not-an-email',
      password: 'SecurePass1!',
      confirmPassword: 'SecurePass1!'
    };
    const options = { isRegistration: true };
    const errors = validateUserData(userData, options);
    expect(errors).toContain('Email format is invalid');
  });

  test('invalid US ZIP code returns specific error', () => {
    const userData = {
      address: {
        street: '123 Main St',
        city: 'Springfield',
        zip: 'INVALID',
        country: 'US'
      }
    };
    const errors = validateUserData(userData, {});
    expect(errors).toContain('Invalid US ZIP code format');
  });

  test('profile update mode does not require password fields', () => {
    const userData = {
      firstName: 'John',
      lastName: 'Doe',
      dateOfBirth: '1990-01-15',
      address: {
        street: '123 Main St',
        city: 'Springfield',
        zip: '12345',
        country: 'US'
      }
    };
    const options = { isRegistration: false };
    expect(validateUserData(userData, options)).toEqual([]);
  });
});
```

---

## Reflection Questions

### How did breaking down the function improve readability and maintainability?

The original function required reading 130 lines to understand one validation rule.
Now, to understand how phone validation works, you read 8 lines in validatePhone().
If a bug is reported about postal codes, you go directly to validatePostalCodeForCountry().
You no longer need to understand the entire system to fix one small part of it.

### What was the most challenging part of decomposing the function?

Deciding where one responsibility ends and another begins.
For example, validateAddress() calls validatePostalCodeForCountry() internally.
I debated whether postal code validation should be inside validateAddress or
called separately from the orchestrator. I chose to keep it inside validateAddress
because postal codes only make sense in the context of an address —
they are not an independent data type in this system.

### Which extracted function would be most reusable?

validateEmail() and validatePostalCodeForCountry() are the most reusable.
Email validation is needed in almost every application that accepts user input.
Postal code validation by country is useful anywhere an address form exists.
Both are now standalone functions that can be imported and used independently
without bringing the entire user validation system with them.

### Benefits gained from this refactoring

1. **Testability:** Each validator can be unit tested in complete isolation.
   Testing validatePassword() no longer requires setting up a full registration form.

2. **Readability:** The orchestrator function now reads like a checklist —
   you can see exactly what gets validated at a glance.

3. **Maintainability:** Adding a new validation rule (e.g., validateProfilePicture)
   means writing one new function and adding one line to the orchestrator.
   In the original, it meant finding the right place in 130 lines of nested code.

4. **Debugging:** When a validation error is reported, the stack trace now
   points directly to the responsible function rather than a line number
   inside a 130-line monolith.

5. **The passwordStrengthRules array:** By replacing 5 separate if/else blocks
   with a data-driven array of rule objects, adding or removing a password
   rule is now a one-line change.