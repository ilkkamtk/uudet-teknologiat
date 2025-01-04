# React Native for Windows + macOS

## Introduction

[React Native for Windows + macOS ](https://microsoft.github.io/react-native-windows/)is a project that adds support for the Windows 10 SDK and macOS 10.12+ to React Native. The project is a fork of the main React Native repository, and it is developed and maintained by Microsoft.

## Why extend React Native to desktop?

- **Code reuse**: React Native allows you to write your app once and run it on multiple platforms. By extending React Native to Windows and macOS, you can target desktop platforms with the same codebase.
   - This is the end goal. Still a new project, so not all features are supported yet.
- **Native performance**: React Native for Windows + macOS uses native components and APIs to provide a native experience on Windows and macOS.
   - This means that RN components have native performance but business logic is still in JS.
   - Performance of fully native apps is still better overall.
- **Leverage existing skills**: If you are already familiar with React Native, you can use the same skills to build desktop apps.
- **Leverage existing code**: If you have an existing React Native app, you can extend it to Windows and macOS without rewriting the entire app.

## Use cases

- Business productivity apps
  - e.g. email clients, chat apps, project management tools
  - commonly used in both mobile and desktop
- Cross-platform tools with desktop needs
   - e.g. developer tools
- Apps that need to integrate with the OS
   - e.g. file system access, system notifications, system tray

## Advantages

- Shared codebase across mobile and desktop. 
- Access to native desktop APIs and UI components. 
- Microsoft’s active support and tooling (for Windows). 
- Leverage React ecosystem with additional desktop support.

## Limitations

- More platform-specific coding than mobile. 
- Smaller community and ecosystem for Windows/macOS compared to iOS/Android. 
- Limited third-party library support for desktop features.


## Platform-specific features

### Windows

- App lifecycle and windowing model
   - Windows apps have different lifecycle events and windowing model compared to mobile.
- [WinUI](https://microsoft.github.io/react-native-windows/docs/0.67/winui3)
  - a native UI framework for Windows apps.

### macOS

- Integration with macOS AppKit
   - macOS apps have different UI components and APIs compared to Windows.
- Using macOS-specific native modules and components
   - e.g. system menu, touch bar, macOS notifications.
- Handling macOS-specific gestures and window behavior
   - e.g. swipe gestures, window resizing.

## Getting started

- Check system requirements and install prerequisites
   - [Windows](https://microsoft.github.io/react-native-windows/docs/rnw-dependencies)
   - [macOS](https://microsoft.github.io/react-native-windows/docs/rnm-dependencies)

### Installation
- [Windows](https://microsoft.github.io/react-native-windows/docs/getting-started)
- [macOS](https://microsoft.github.io/react-native-windows/docs/rnm-getting-started)

## Exercise

Here’s the updated assignment with all code written in **TypeScript**:

---

### **Assignment Instructions: Create a React Native Application for Customizing a REST API Project (TypeScript)**

---

#### **Objective:**
Develop a **React Native application** for **Windows** or **macOS** that helps developers quickly clone, customize, and initialize a REST API project from a GitHub repository. The app must:
1. Clone a predefined REST API project repository from GitHub.
2. Customize the project using settings entered via a form.
3. Generate a TypeScript-based model, controller, and route as examples.

---

### **Assignment Requirements**

#### **1. Application Overview**
The app should:
- Target **Windows** or **macOS** (your choice).
- Use a form-based interface to gather information about the project, such as:
    - **Project Name**
    - **Description**
    - **Database Table Name** (e.g., `users`).
    - **Fields** (name, type, and constraints for the database table).
- Clone an example REST API project from a GitHub repository.
- Customize the cloned project by:
    - Renaming the folder to the project name.
    - Updating the `package.json` with the project name and description.
    - Generating new **TypeScript** files for a model, controller, and route based on the provided table name and fields.

---

#### **2. Features and Functionalities**

##### **Frontend (React Native UI)**
- A form with input fields for:
    - **Project Name**
    - **Description**
    - **Database Table Name**
    - **Fields** (dynamic list of inputs where users can add field name, type, and constraints).
- A "Generate Project" button to trigger the cloning and customization process.
- Feedback section to show progress (e.g., "Cloning repository...", "Generating model and route...").

---

#### **3. Technical Workflow**

##### **1. Clone the Repository**
- Clone the example REST API repository from GitHub using the `git clone` command:
  ```bash
  git clone https://github.com/example/rest-api-template.git <project-name>
  ```  

##### **2. Customize the Project**
- Rename the cloned folder to the project name entered in the form.
- Update the `package.json` with the project name and description:
  ```json
  {
    "name": "<project-name>",
    "version": "1.0.0",
    "description": "<description>",
    "main": "src/index.ts",
    "scripts": {
      "start": "ts-node src/index.ts",
      "dev": "nodemon src/index.ts"
    },
    "dependencies": {
      "express": "^4.x",
      "cors": "^2.x",
      "helmet": "^6.x",
      "dotenv": "^16.x",
      "sqlite3": "^5.x",
      "sequelize": "^6.x"
    },
    "devDependencies": {
      "typescript": "^5.x",
      "ts-node": "^10.x",
      "nodemon": "^2.x"
    }
  }
  ```  

##### **3. Generate Example TypeScript Files**

- **Model (TypeScript)**
  ```typescript
  // src/models/<table-name>.ts
  import { DataTypes, Model, Sequelize } from 'sequelize';
  import sequelize from '../config/database';

  interface <TableName>Attributes {
    id: number;
    <FieldDeclarations>
  }

  class <TableName> extends Model<<TableName>Attributes> {}

  <TableName>.init(
    {
      id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true,
      },
      <FieldDefinitions>
    },
    {
      sequelize,
      modelName: '<TableName>',
    }
  );

  export default <TableName>;
  ```  

  **Example for a `users` table**:
  ```typescript
  import { DataTypes, Model } from 'sequelize';
  import sequelize from '../config/database';

  interface UserAttributes {
    id: number;
    name: string;
    email: string;
  }

  class User extends Model<UserAttributes> {}

  User.init(
    {
      id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true,
      },
      name: {
        type: DataTypes.STRING,
        allowNull: false,
      },
      email: {
        type: DataTypes.STRING,
        unique: true,
      },
    },
    {
      sequelize,
      modelName: 'User',
    }
  );

  export default User;
  ```  

- **Controller (TypeScript)**
  ```typescript
  // src/controllers/<table-name>.controller.ts
  import { Request, Response } from 'express';
  import <TableName> from '../models/<table-name>';

  export const getAll = async (req: Request, res: Response): Promise<void> => {
    try {
      const data = await <TableName>.findAll();
      res.json(data);
    } catch (error) {
      res.status(500).send(error.message);
    }
  };

  export const create = async (req: Request, res: Response): Promise<void> => {
    try {
      const newRecord = await <TableName>.create(req.body);
      res.status(201).json(newRecord);
    } catch (error) {
      res.status(500).send(error.message);
    }
  };
  ```  

- **Route (TypeScript)**
  ```typescript
  // src/routes/<table-name>.routes.ts
  import { Router } from 'express';
  import { getAll, create } from '../controllers/<table-name>.controller';

  const router: Router = Router();

  router.get('/', getAll);
  router.post('/', create);

  export default router;
  ```  

##### **4. Update the App Entry Point (TypeScript)**
- Modify the `src/index.ts` file to include the new route:
  ```typescript
  import express from 'express';
  import cors from 'cors';
  import helmet from 'helmet';
  import <TableName>Routes from './routes/<table-name>.routes';

  const app = express();

  app.use(cors());
  app.use(helmet());
  app.use(express.json());

  app.use('/<table-name>', <TableName>Routes);

  const PORT = process.env.PORT || 3000;
  app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
  ```  

---

#### **4. Submission Checklist**
- The React Native app clones the example REST API repository and customizes it with TypeScript.
- The project includes a TypeScript-based model, controller, and route based on form input.
- Logs and error handling are implemented in the app.
- Include a **README** file in the generated project with instructions on how to set up and run the project.
- Submit the React Native source code and a **README** file explaining how to run your app.

---

### **Assessment Criteria**

| **Criteria**                          | **Points** |  
|---------------------------------------|------------|  
| UI Design and Functionality           | 20         |  
| Repository Cloning and Customization  | 30         |  
| Correct Generation of TypeScript Files| 30         |  
| Logs and Error Handling               | 10         |  
| Submission Completeness               | 10         |  
| **Total**                             | **100**    |  

---

### **Deadline**
Submit your assignment by **[insert deadline]** via the course submission platform.

---

Feel free to ask questions or seek clarification during office hours or via the discussion board. Good luck! 🚀
