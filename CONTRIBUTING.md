# ❤️ We Love Contributions!

Thank you for your interest in making DevOps Cheatsheets better! Your help is what makes this project a great resource.

## How to Contribute

There are three main ways to contribute:

1. **[Fixing a Typo or Error](#1-fixing-a-typo-or-error)**
2. **[Suggesting Content for an Existing Cheatsheet](#2-suggesting-content)**
3. **[Adding a New Cheatsheet](#3-adding-a-new-cheatsheet)**

---

### 1. Fixing a Typo or Error

Found a mistake? The fastest way to fix it is:

1. Click the "pencil" icon ✏️ on the top right of the file you want to edit.
2. Make your change directly in the GitHub editor.
3. At the bottom, type a clear commit message (e.g., "Fix: Corrected typo in docker run command").
4. Click "Propose changes" and submit your Pull Request!

### 2. Suggesting Content

Have an idea for a command or concept that's missing from a sheet?

1. Go to the "Issues" tab.
2. Open a new issue using the **"Content Suggestion"** template.
3. Clearly explain what you want to add and why it's important.

### 3. Adding a New Cheatsheet

Want to add a cheatsheet for a tool we don't have yet (e.g., "Vagrant")?

1. **Open an Issue First:** Open a new issue using the **"New Cheatsheet"** template. This lets us discuss the new sheet and make sure it fits the project scope.
2. **Fork the Repository:** Click the "Fork" button at the top right.
3. **Clone Your Fork:** `git clone https://github.com/YOUR_USERNAME/DevOps-Cheatsheets.git`
4. **Create a New Branch:** `git checkout -b feat/add-vagrant-cheatsheet`
5. **Create the New Folder:**
   * Find the right category (e.g., `06-Infrastructure-as-Code/`).
   * Create a new folder for your tool (e.g., `06-Infrastructure-as-Code/vagrant/`).
   * Inside that folder, create a `README.md` file.
6. **Write Your Cheatsheet:**
   * **CRITICAL:** Copy the template from an existing cheatsheet (like `03-Containerization/docker/README.md`) to ensure you follow the correct format.
   * **The format (Concepts, Commands, Gotchas) must be consistent.**
7. **Add to Main `README.md`:** Add a link to your new cheatsheet in the main `README.md` file's Table of Contents.
8. **Commit and Push:**
   * `git add .`
   * `git commit -m "Feat: Add Vagrant cheatsheet"`
   * `git push origin feat/add-vagrant-cheatsheet`
9. **Open a Pull Request:** Go to the main repository and you will see a prompt to open a PR from your new branch.

---

## 📝 Style Guidelines

* Use clear, concise language.
* Follow the existing format for consistency.
* Test all commands before submitting.
* Include examples where helpful.
* Use proper Markdown formatting.

## 🐛 Reporting Issues

If you find a bug or have a suggestion:

1. Check if the issue already exists.
2. If not, create a new issue with a clear title and description.
3. Include steps to reproduce (if applicable).

## 📜 Code of Conduct

Please note that this project is released with a [Code of Conduct](CODE_OF_CONDUCT.md). By participating in this project you agree to abide by its terms.

---

Thank you for making this resource better for everyone! 🙏
