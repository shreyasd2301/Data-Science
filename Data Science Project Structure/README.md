# Data Science Project Structure

## Why use a specific project structure in Git?

### Take care of teammates

A well-defined, standard project structure means that a newcomer can begin to understand an analysis without digging into extensive documentation. It also means that they don't necessarily have to read 100% of the code before knowing where to look for very specific things.

Well organized code tends to be self-documenting in that the organization itself provides context for your code without much overhead. Other teammates will thank you for this because they can:

- Collaborate more easily with you on this analysis
- Learn from your analysis about the process and the domain
- Feel confident in the conclusions at which the analysis arrives

### Don't forget about yourself

Ever tried to reproduce an analysis that you did a few months ago or even a few years ago? You may have written the code, but it's now impossible to decipher whether you should use `make_figures.py.old`, `make_figures_working.py` or `new_make_figures01.py` to get things done. Here are some questions we've learned to ask with a sense of existential dread:

- Are we supposed to go in and join column X to the data before we get started or did that come from one of the notebooks?
- Come to think of it, which notebook do we have to run first before running the plotting code: was it "process data" or "clean data"?
- Where did the shapefiles get downloaded for the geographic plots?
- Et cetera…

## Creating a good project structure

### Automating project template creation with [Cookiecutter Data Science](https://github.com/drivendata/cookiecutter-data-science)

There is not a clear consensus in the data science community on best practices for organizing machine learning projects, but there is a tool called Cookiecutter Data Science which is a [standardized but flexible project structure for doing and sharing data science work. A few lines of code set up a whole series of subdirectories and make it easier to start, structure, and share analysis](https://github.com/drivendata/cookiecutter-data-science).

## Data is immutable, add `.gitignore` file

Don't ever edit your raw data, especially not manually, and especially not in Excel. Don't overwrite your raw data. Don't save multiple versions of the raw data. Treat the data (and its format) as immutable. The code you write should move the raw data through a pipeline to your final analysis. You shouldn't have to run all of the steps every time you want to make a new figure, but anyone should be able to reproduce the final products with only the code in src and the data.

Also, if data is immutable, it doesn't need source control in the same way that code does. Therefore, **by default, the data folder is included in the `.gitignore` file**. If you have a small amount of data that rarely changes, you may want to include the data in the repository.  Some other options for storing/syncing large data include  AWS S3 with a syncing tool (e.g., s3cmd), Git Large File Storage, Git Annex, and dat.

## Data Analysis

Often in an analysis you have long-running steps that preprocess data or train models. If these steps have been run already (and you have stored the output somewhere like the data/interim directory), you don't want to wait to rerun them every time. We prefer make for managing steps that depend on each other, especially the long-running ones. Make is a common tool on Unix-based platforms (and is available for Windows). Following the [make documentation](https://www.gnu.org/software/make/), [Makefile conventions](https://www.gnu.org/prep/standards/html_node/Makefile-Conventions.html#Makefile-Conventions), and [portability guide](http://www.gnu.org/savannah-checkouts/gnu/autoconf/manual/autoconf-2.69/html_node/Portable-Make.html#Portable-Make) will help ensure your Makefiles work effectively across systems. Here are [some](http://zmjones.com/make/) [examples](http://blog.kaggle.com/2012/10/15/make-for-data-scientists/) to [get started](https://web.archive.org/web/20150206054212/http://www.bioinformaticszen.com/post/decomplected-workflows-makefiles/).

There are other tools for managing DAGs that are written in Python instead of a DSL (e.g., [Paver](http://paver.github.io/paver/#), [Luigi](http://luigi.readthedocs.org/en/stable/index.html), [Airflow](https://airflow.apache.org/index.html), [Snakemake](https://snakemake.readthedocs.io/en/stable/), [Ruffus](http://www.ruffus.org.uk/), or [Joblib](https://pythonhosted.org/joblib/memory.html)). Feel free to use these if they are more appropriate for your analysis.

## 🏗️ Build from the environment up

The first step in reproducing an analysis is always reproducing the computational environment it was run in. You need the same tools, the same libraries, and the same versions to make everything play nicely together.

One effective approach to this is to use [virtualenv](https://virtualenv.pypa.io/en/latest/) (we recommend [virtualenvwrapper](https://virtualenvwrapper.readthedocs.org/en/latest/) for managing virtualenvs). By listing all of your requirements in the repository (we include a `requirements.txt` file) you can easily track the packages needed to recreate the analysis. Here is a good workflow:

1. Run `mkvirtualenv` when creating a new project
2. `pip install` the packages that your analysis needs
3. Run `pip freeze > requirements.txt` to pin the exact package versions used to recreate the analysis
4. If you find you need to install another package, run `pip freeze > requirements.txt` again and commit the changes to version control.

If you have more complex requirements for recreating your environment, consider a virtual machine-based approach such as [Docker](https://www.docker.com/) or [Vagrant](https://www.vagrantup.com/). Both of these tools use text-based formats (Dockerfile and Vagrantfile respectively) you can easily add to source control to describe how to create a virtual machine with the requirements you need.

## 🔐 Store your secrets and config variables in a special file

You really don't want to leak your AWS secret key or Postgres username and password on  Git. Enough said — see the [Twelve Factor App](http://12factor.net/config) principles on this point. Here's one way to do this:

### 🔑 Store your secrets and config variables in a special file

Create a `.env` file in the project root folder. Thanks to the `.gitignore`, this file should never get committed into the version control repository. Here's an example:

```
# example .env file
DATABASE_URL=postgres://username:password@localhost:5432/dbname
AWS_ACCESS_KEY=myaccesskey
AWS_SECRET_ACCESS_KEY=mysecretkey
OTHER_VARIABLE=something
```

### 📄 Use a package to load these variables automatically

If you look at the stub script in `src/data/make_dataset.py` from the project template created by Cookiecutter Data Science, it uses a package called [python-dotenv](https://github.com/theskumar/python-dotenv) to load up all the entries in this file as environment variables so they are accessible with os.environ.get. Here's an example snippet adapted from the python-dotenv documentation:

```@python
# src/data/dotenv_example.py
import os
from dotenv import load_dotenv, find_dotenv

# find .env automagically by walking up directories until it's found
dotenv_path = find_dotenv()

# load up the entries as environment variables
load_dotenv(dotenv_path)

database_url = os.environ.get("DATABASE_URL")
other_variable = os.environ.get("OTHER_VARIABLE")
```

### AWS CLI configuration

When using Amazon S3 to store data, a simple method of managing AWS access is to set your access keys to environment variables. However, managing mutiple sets of keys on a single machine (e.g. when working on multiple projects) it is best to use a [credentials file](https://docs.aws.amazon.com/cli/latest/userguide/cli-config-files.html), typically located in `~/.aws/credentials`. A typical file might look like:

```
[default]
aws_access_key_id=myaccesskey
aws_secret_access_key=mysecretkey

[another_project]
aws_access_key_id=myprojectaccesskey
aws_secret_access_key=myprojectsecretkey
```

## 🔩 Tools

- **Cookiecutter Data Science:**
  - [Cookiecutter Data Science](http://drivendata.github.io/cookiecutter-data-science/)
  - :octocat:[cookiecutter-data-science](https://github.com/drivendata/cookiecutter-data-science)
- [readme.so](https://readme.so/)

### Other tools

- [Filesystem Hierarchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard) for Unix-like systems
- [Git Large File Storage](https://git-lfs.github.com)
- [Git Annex](https://git-annex.branchable.com/)
- [make documentation](https://www.gnu.org/software/make/)
- [Makefile conventions](https://www.gnu.org/prep/standards/html_node/Makefile-Conventions.html#Makefile-Conventions)
- [portability guide](http://www.gnu.org/savannah-checkouts/gnu/autoconf/manual/autoconf-2.69/html_node/Portable-Make.html#Portable-Make)
- [GNU Make for Reproducible Data Analysis](http://zmjones.com/make/)
- []()

## 🔗 Reference to articles to read

- [Automate your data science project structure in three easy steps](https://towardsdatascience.com/automate-your-data-science-project-structure-in-three-easy-steps-277c92328d24)
- [How to Structure a Data Science Project for Readability and Transparency](https://towardsdatascience.com/how-to-structure-a-data-science-project-for-readability-and-transparency-360c6716800)
  - [Data Science Cookie Cutter](https://github.com/khuyentran1401/data-science-template)
- [How to Create a Professional Github Data Science Repository](https://towardsdatascience.com/how-to-create-a-professional-github-data-science-repository-84e9607644a2)