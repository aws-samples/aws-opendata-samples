
# Creating a Multilanguage Registry Page

## Overview

This guidance provides prescriptive steps to create a Registry page (yaml) with more than 1 language that is compatible with AWS Data Exchange integration. **[RCM CEOS Analysis Read Data](https://registry.opendata.aws/rcm-ceos-ard/)** is a good example to follow.

---

## YAML Configuration

### Step 1: Create the yaml in English

1. Browse to the [AWS Open Data github repository](https://github.com/awslabs/open-data-registry).
   
2. Click on **Code** button, then copy the HTTPS URL to the clipboard.

3. On your computer, create a new directory:
```mkdir mydirectory```

4. Open the new directory:
```cd mydirectory```

5. Paste this command to clone the AWS Open Data github repo:
```git clone https://github.com/awslabs/open-data-registry.git```

6. Open the open-data-registry folder:
```cd open-data-registry```

7. Open the datasets folder:
```cd datasets```

8. Use nano or similar code editor to add your yaml:
```nano mydataset.yaml```

9. Enter your dataset information following the yaml file structure listed on the Open Data github page: https://github.com/awslabs/open-data-registry.  You will enter this information in English to start.

---

### Step 2: Add one or more additional languages

1. Start with the dataset title or **Name** field in the yaml. After the English version of the title, enter a pipe character "|" and then add the second language. **[RCM CEOS Analysis Read Data](https://registry.opendata.aws/rcm-ceos-ard/)** is a good example to follow.
```RCM CEOS Analysis Ready Data | Données prêtes à l'analyse du CEOS pour le MCR```

2. For **Description**, after the English description, enter 2 breaks:
```<br/>```
```<br/>``` 

and then enter the second language.  Continue this process for all languages.

---

### Step 3: Submit your changes

1. Copy this command to verify your changes:
```git status```

It should say that you have an untracked file, which is your new yaml.

2. Copy this command and rename the FILENAME to your new yaml:
```git add FILENAME.yaml```

3. Copy this command to verify your new yaml is ready to be committed:
```git status```

4. Copy this command change MESSAGE to a brief explanation about your change, such as *adding new FOO yaml*:
```git commit -m "MESSAGE"```

5. Create a Pull Request to submit your change, again replacing MESSAGE with your brief explanation about your change:
```gh pr create --title "MESSAGE"```

---

Once submitted, the Open Data team will review your pull request. If there are any questions or updates required, we’ll leave comments. Once approved and merged, your SNS topic will appear in the dataset entry on RODA.

**Thank you for improving the discoverability and usability of your dataset!**
For questions or feedback, email us at opendata@amazon.com.
