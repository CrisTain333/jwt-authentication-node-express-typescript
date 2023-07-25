Today We Will Learn How to Build NodeJs Authentication API using JWT, Express, Typescript .

Table of contents

- What Is JSON Web Token (JWT)
- Initialize Project
- Install dependencies and devDependencies
- Setup express server with typescript
- Create User Model
- Create User Api To Register
- Crate Login Api and Implement JWT Authentication

Github Repo :

## 1. What Is JSON Web Token (JWT)

**JSON web token(JWT)** is JSON Object which is used to securely transfer information over the web(between two parties). It is generally used for authentication systems and can also be used for information exchange.

This is used to transfer data with encryption over the internet also these tokens can be more secured by using an additional signature.

## 2. Initialize Project

```
mkdir jwt-authentication
cd jwt-authentication
npm init --yes
```

## 3. Install dependencies and devDependencies

3.1 Install dependencies

```
npm install express mongoose cors jsonwebtoken dotenv
```

3.2 Install devDependencies

```
npm install -D typescript nodemon @types/express @types/cors @types/jsonwebtoken
```

3.3 Add a tsconfig.json for typescript configuration

```
tsc --init
```

- add this cofig in tsconfig.json file

```
{
    "compilerOptions": {
        /* Language and Environment */
        "target": "es2016" /* Set the JavaScript language version for emitted JavaScript and include compatible library declarations. */,
        /* Modules */
        "module": "commonjs" /* Specify what module code is generated. */,
        "rootDir": "./",
        "outDir": "./dist",
        "esModuleInterop": true,
        "forceConsistentCasingInFileNames": true /* Ensure that casing is correct in imports. */,

        /* Type Checking */
        "strict": true,
        "skipLibCheck": true /* Skip type checking all .d.ts files. */
    }
}

```

## 4. Setup express server with typescript

In the root create a file name app.ts

![folder_structure](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/miyecut0ap8dsmpu7inl.png)

```
import express from "express";
import { Application } from "express";
import mongoose from "mongoose";
import cors from "cors";
import dotenv from "dotenv";
// Create the express app and  import the type of app from express;
const app: Application = express();

// Cors
app.use(cors());
//configure env;
dotenv.config();
// Parser
app.use(express.json());
app.use(
  express.urlencoded({
    extended: true,
  })
);
// Declare The PORT Like This
const PORT: number = 8000;

app.get("/", (req, res) => {
  res.send("<h1>Welcome To JWT Authentication </h1>");
});

// Listen the server
app.listen(PORT, async () => {
  console.log(`🗄️  Server Fire on http:localhost//${PORT}`);

  // Connect To The Database
  try {
    await mongoose.connect(
      process.env.DATABASE_URL as string
    );
    console.log("🛢️  Connected To Database");
  } catch (error) {
    console.log("⚠️ Error to connect Database");
  }
});

```

In the Package.json add this srcipt

```
"scripts": {
    "dev": "nodemon app.ts",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
```

After Update the package.json file open terminal and run the command

```
npm run dev
```

In the Terminal You will see

![Termail_image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/at0yxvgw4guomp2grpm6.png)

Open Your Browser and type the url "http://localhost:8000"

![browser_image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6wrvr5m14gsd8v5qpyh3.png)

The Express server is now up and running!

## 5. Create User Model

The User model is created using using the mongoose package. This model represent each user in the database.

In the root create a models folder, create the file user.ts .

![user_model](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qn3u4fmux4kmqb9g6aiw.png)

inside user.ts create the user schema;

```
import mongoose from "mongoose";

const UserSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: true,
    },
    email: {
      type: String,
      unique: true,
      required: true,
    },
    password: {
      type: String,
      required: true,
    },
  },
  { timestamps: true }
);

export const User = mongoose.model("Users", UserSchema);

```

After creating the user model we are ready to implement the jwt Authentication.

## 6.Create User Api To Register

in the "app.ts" create a register api to create account.
api = '/auth/register'

```
app.post("/auth/register", async (req, res) => {
  try {
    // ** Get The User Data From Body ;
    const user = req.body;

    // ** destructure the information from user;
    const { name, email, password } = user;

    // ** Check the email all ready exist  in database or not ;
    // ** Import the user model from "./models/user";

    const isEmailAllReadyExist = await User.findOne({
      email: email,
    });

    // ** Add a condition if the user exist we will send the response as email all ready exist
    if (isEmailAllReadyExist) {
      res.status(400).json({
        status: 400,
        message: "Email all ready in use",
      });
       return;
    }

    // ** if not create a new user ;
    // !! Don't save the password as plain text in db . I am saving just for demonstration.
    // ** You can use bcrypt to hash the plain password.

    // now create the user;
    const newUser = await User.create({
      name,
      email,
      password,
    });

    // Send the newUser as  response;
    res.status(200).json({
      status: 201,
      success: true,
      message: " User created Successfully",
      user: newUser,
    });
  } catch (error: any) {
    // console the error to debug
    console.log(error);

    // Send the error message to the client
    res.status(400).json({
      status: 400,
      message: error.message.toString(),
    });
  }
});
```

Now test the api. I am using ThunderClient Extension available in vs code Market Place.

if everything works well you will get a success message like this

![register_api_test](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bqvely5yp3lr77dbpl32.png)

now if i try to create the same user with the same email it will send me a error message.

![register_aapi_eror_message](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wxjnclfx7ig7gqqao40g.png)

after creating the user we can start jwt implement .

## 7. Crate Login Api and Implement JWT Authentication

api = '/auth/login'

- Get The User Data From Body .
- destructure the information from user.
- Check the (email/user) exist in database or not .
- if there is not any user with the email we send user not found.
- if the (user) exist in database we will check the password is valid or not .
- compare the password in database and the password in the request body.
- if not matched send response that wrong password.
- if the email and password is valid create a token .
- To create a token JsonWebToken (JWT) receive's 3 parameter

  1. Payload - This contains the claims or data you want to include in the token.
  2. Secret Key - A secure key known only to the server used for signing the token.
  3. expiration - Additional settings like token expiration or algorithm selection.

- Don't Provide the secret openly, This secret is very sensitive for the server . keep it in the .env file. I am Keeping it Open just for demonstration.

- After creating the token send the response.

```
app.post("/auth/login", async (req, res) => {
  try {
    // ** Get The User Data From Body ;
    const user = req.body;

    // ** destructure the information from user;
    const { email, password } = user;

    // ** Check the (email/user) exist  in database or not ;
    const isUserExist = await User.findOne({
      email: email,
    });

    // ** if there is not any user we will send user not found;
    if (!isUserExist) {
      res.status(404).json({
        status: 404,
        success: false,
        message: "User not found",
      });
return;
    }

    // ** if the (user) exist  in database we will check the password is valid or not ;
    // **  compare the password in db and the password sended in the request body

    const isPasswordMatched =
      isUserExist?.password === password;

    // ** if not matched send response that wrong password;

    if (!isPasswordMatched) {
      res.status(400).json({
        status: 400,
        success: false,
        message: "wrong password",
      });
        return;
    }

    // ** if the email and password is valid create a token

    /*
    To create a token JsonWebToken (JWT) receive's 3 parameter
    1. Payload -  This contains the claims or data you want to include in the token.
    2. Secret Key - A secure key known only to the server used for signing the token.
    3. expiration -  Additional settings like token expiration or algorithm selection.
    */

    // !! Don't Provide the secret openly, keep it in the .env file. I am Keeping Open just for demonstration

    // ** This is our JWT Token
    const token = jwt.sign(
      { _id: isUserExist?._id, email: isUserExist?.email },
      "YOUR_SECRET",
      {
        expiresIn: "1d",
      }
    );

    // send the response
    res.status(200).json({
      status: 200,
      success: true,
      message: "login success",
      token: token,
    });
  } catch (error: any) {
    // Send the error message to the client
    res.status(400).json({
      status: 400,
      message: error.message.toString(),
    });
  }
});
```

Now test the api. I am using ThunderClient Extension available in vs code Market Place.

Now If Try To Login With Wrong Email that it will send the error of user not found

![login_wrong_email_response](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/iv8y0xoo2pvaqq4hrg3u.png)

Now If Try To Login With right Email but wrong password it will send error of wrong password.

![login_wrong_password_response](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n90zy6odt49h1s216gab.png)

If I Provide The Correct Email and Password it will send me the token that we created.

![login_seccess_response](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tvzherpnl57ixrga3h8d.png)

Now if we copy the token that we got after the login and go to https://jwt.io/ and past the token and press decode

we can see the information that we provided on creation time .

![json_decode](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bgn5eydygqqsydp3t6bd.png)

If You Found This Repo Help full , Share With Other's .

## That’s all.
