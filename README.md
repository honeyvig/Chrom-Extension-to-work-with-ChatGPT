# Chrom-Extension-to-work-with-ChatGPT
I need a Chrome Extension to be developed as per the following specifications:

1. The extension is similar to the one available here:

a. https://chrome.google.com/webstore/detail/chatonai-unlock-the-power/feeonheemodpkdckaljcjogdncpiiban?hl=en
b. https://appsumo.com/products/copilotly/

2. The extension provides prompt templates that can be used with ChatGPT. The user will be able to use the templates as they are, or make modifications and send templated prompts to ChatGPT.
I will provide the templates to be set up in the Extension.

3. Attached is a word document that contains some sample prompt templates. There will around 350 templates in total, arranged into about 20 categories. I will provide all the 350 templates to you in word document, after I award you the project.

4. I need you to publish the final extension on Chrome Web Store by creating a Developer account for me. I will also need you to create an Admin Panel which will let me manage (Edit, Delete, Add) the users, and prompt templates.

5. Once the extension is published on Chrome Web Store, I need you to also set up the Licensing module so that only authorized users can access and install the extension.

Important
a. I want you to first go through the "Chatonai" chrome extension and thoroughly understand it's layout, features, and functionality before applying to this project.
b. This is the first of a series of projects related to creating multiple Chrome extensions. Once you establish your credibility by completing this project successfully to my satisfaction, we will discuss the next set of projects too.
-----
To develop the Chrome extension you’ve requested, we will break down the project into its key components and discuss how to implement each one. Here's an outline of the approach and code structure for creating the extension:
1. Overview of the Extension

The extension will allow users to access prompt templates for ChatGPT, customize them, and send them to ChatGPT. The functionality of the extension will include:

    Displaying a list of templates, grouped by categories.
    Allowing users to edit the templates and send them to ChatGPT.
    Managing user access and licensing.
    A simple Admin Panel to manage templates and users.

2. Project Breakdown
2.1 Understanding the Existing Extensions

You mentioned two extensions that serve as inspiration: Chatonai and Copilotly. Before diving into the code, we’ll need to analyze these extensions’ layout, features, and functionality. Here's what I can infer from the description:

    Prompt Templates: Both extensions offer pre-defined templates to help users interact with ChatGPT more effectively.
    User Interface (UI): A well-organized and intuitive UI to browse through templates.
    License Management: A mechanism that ensures only authorized users can access the extension.
    Admin Panel: An interface for managing users and templates.

2.2 Extension Components
1. Manifest File (manifest.json)

This is the configuration file that tells Chrome how to load your extension.

{
  "manifest_version": 3,
  "name": "ChatGPT Prompt Templates",
  "description": "Access and manage your ChatGPT prompt templates.",
  "version": "1.0",
  "permissions": [
    "storage",
    "activeTab",
    "identity",
    "webNavigation",
    "cookies"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["https://chat.openai.com/*"],
      "js": ["content.js"]
    }
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "assets/icon.png",
      "48": "assets/icon.png",
      "128": "assets/icon.png"
    }
  },
  "options_page": "options.html",
  "icons": {
    "16": "assets/icon.png",
    "48": "assets/icon.png",
    "128": "assets/icon.png"
  }
}

2. Background Script (background.js)

The background script will handle user authentication and licensing logic, interacting with your server to verify license validity.

// background.js
chrome.runtime.onInstalled.addListener(() => {
  console.log("ChatGPT Prompt Templates Extension Installed");
  // Here you can initialize the template storage and manage users
});

// Add listener to check user license
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === 'checkLicense') {
    // Check license from server or local storage
    let isAuthorized = checkLicense();
    sendResponse({ isAuthorized });
    return true;
  }
});

function checkLicense() {
  // Check license from backend or local storage
  return true; // Assume license is valid for now
}

3. Popup (popup.html)

The popup will display the categories and templates for the user to browse and use. It will allow users to select a template, modify it, and send it to ChatGPT.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ChatGPT Templates</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <div id="container">
    <h1>ChatGPT Prompt Templates</h1>
    <div id="categories"></div>
    <div id="templates"></div>
    <textarea id="modified-prompt" rows="5" cols="40"></textarea>
    <button id="send-to-chatgpt">Send to ChatGPT</button>
  </div>
  <script src="popup.js"></script>
</body>
</html>

4. Popup Script (popup.js)

This file will load templates and categories dynamically from the background script or a stored JSON file. Users will also be able to modify templates before sending them to ChatGPT.

// popup.js

// Load categories and templates on popup open
document.addEventListener("DOMContentLoaded", () => {
  chrome.runtime.sendMessage({ type: 'getTemplates' }, (response) => {
    if (response.templates) {
      loadTemplates(response.templates);
    }
  });
});

function loadTemplates(templates) {
  const categoriesContainer = document.getElementById("categories");
  const templatesContainer = document.getElementById("templates");

  // Group templates by categories
  const categories = Object.keys(templates);
  
  categories.forEach(category => {
    const categoryDiv = document.createElement("div");
    categoryDiv.classList.add("category");
    categoryDiv.innerText = category;
    categoriesContainer.appendChild(categoryDiv);
    
    const templateList = document.createElement("ul");
    templates[category].forEach(template => {
      const listItem = document.createElement("li");
      listItem.innerText = template.name;
      listItem.addEventListener("click", () => {
        document.getElementById("modified-prompt").value = template.prompt;
      });
      templateList.appendChild(listItem);
    });
    
    categoriesContainer.appendChild(templateList);
  });
}

// Send the modified prompt to ChatGPT
document.getElementById("send-to-chatgpt").addEventListener("click", () => {
  const prompt = document.getElementById("modified-prompt").value;
  if (prompt) {
    chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
      chrome.tabs.sendMessage(tabs[0].id, { type: 'sendPrompt', prompt: prompt });
    });
  }
});

5. Content Script (content.js)

The content script will run on the ChatGPT page and handle sending the modified prompt to ChatGPT.

// content.js
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === 'sendPrompt') {
    const prompt = message.prompt;
    // Logic to inject the prompt into the ChatGPT interface
    document.querySelector(".chat-input").value = prompt;
    document.querySelector(".submit-button").click();
  }
});

6. Admin Panel (Options Page) (options.html)

For the Admin Panel, you will need to create a separate page to manage templates and users. The admin panel will allow you to add, edit, and delete templates.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Admin Panel</title>
</head>
<body>
  <h1>Admin Panel</h1>
  <button id="add-template">Add Template</button>
  <button id="delete-template">Delete Template</button>
  <div id="templates-list"></div>
  
  <script src="admin.js"></script>
</body>
</html>

7. Admin Panel Script (admin.js)

The admin script will manage template additions, deletions, and user management.

// admin.js

document.getElementById("add-template").addEventListener("click", () => {
  const templateName = prompt("Enter template name:");
  const templateContent = prompt("Enter template content:");
  if (templateName && templateContent) {
    // Add the template to local storage or backend
    chrome.storage.local.set({ [templateName]: templateContent });
  }
});

document.getElementById("delete-template").addEventListener("click", () => {
  const templateName = prompt("Enter template name to delete:");
  if (templateName) {
    chrome.storage.local.remove(templateName);
  }
});

3. Licensing and User Authentication

To manage licensing, you can implement a simple backend that verifies user credentials and stores the license status. You could use an API or an authentication service (like Firebase Authentication) to validate users. You will need to modify the background script to communicate with the backend for license validation.
4. Publishing to Chrome Web Store

    Create Developer Account: Sign up for a developer account in the Chrome Web Store.
    Upload the Extension: Zip your extension files and upload them to the Chrome Web Store developer dashboard.
    Publish and Setup License: After the extension is published, you can manage users and licensing through your backend.

Conclusion

This outline gives you the overall structure of the Chrome extension, including template management, user interface, and licensing features. Once the backend for licensing and user management is set up, the extension will be fully functional.
