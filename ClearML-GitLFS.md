### Prerequisites

1. Git Repository created and Git LFS enabled on the repo
   
``` 
git clone https://git.web.boeing.com/<project>/myrepo.git
# Zip is the extention that GitLFS uses to package up the files you add to your dataset.
git lfs track "*.zip"
git lfs track "*.np"
git add .gitattributes
git commit -m "adding and setting up git attributes" 
 ```

## Installation

Currently the code is in the following location: 

* BEN: https://git.web.boeing.com/kevin.vasko/clearml.git
* BSF: https://gitlab.gus.bsf.tools/machine-learning/clearml.git


```
git clone <repo>
cd clearml
git switch clearml-gitlfs
python -m venv venv
pip install -r requirements.txt
pip install .
```
The last command will install clearml with its dependencies for the changes with GitLFS.

Add the following configuration section to your `clearml.conf`.

```
    git {
        token_name: "TokenNameFromGitLAb"
        token: "AccessTokenFromGitLab"   
    }
```

Example:
```
...

    azure.storage {
        # max_connections: 2
        # containers: [
        #     {
        #         account_name: "clearml"
        #         account_key: "secret"
        #         # container_name:
        #     }
        # ]
    }
    git {
        token_name: "TokenNameFromGitLAb"
        token: "AccessTokenFromGitLab"   
    }
    log {
        # debugging feature: set this to true to make null log propagate messages to root logger (so they appear in stdout)
        null_log_propagate: false
        task_log_buffer_capacity: 66

        # disable urllib info and lower levels
        disable_urllib3_info: true
    }
...
```

## Usage

The SDK and ClearML CLI should function the same as with S3/Azure/HTTP etc.

Example with CLI:
```
clearml-data create --project GitLFSTest --name gitlfstest
clearml-data add --files yourfile.png
clearml-data add --files yourfile2.png
clearml-data upload --storage git://git.web.boeing.com/myproject/myrepo.git
clearml-data close
```

The format of the `--storage` parameter is special. It must be in the format `git://`. 
Take the `https://` clone command and simply replace the `https` with `git`.

The ClearML SDK functions the same.

```python
# create example dataset
from clearml import StorageManager, Dataset, Task

# # Create a dataset with ClearML`s Dataset class
dataset = Dataset.create(
     dataset_project="GitLFSTest", dataset_name="gitlfstest"
)
### add the example png
dataset.add_files(path=".\\data\\file2.png")
# #
# # # # Upload dataset to ClearML server (customizable)
dataset.upload(output_url="git://git.web.boeing.com/myproject/myrepo.git")
# # #
#### commit dataset changes
dataset.finalize()
```

### Background information 

1. Due to the rush to get this working for the MVP, the delete function is not implement. 
2. Because of the way ClearML works, it must copy the data into a cloned version of your repo so you will have multiple copies of the data on your system.
3. The clone repo lives in a temporary directory. On Windows `C:\Users\<User>\AppData\Local\Temp\clearml-gittmp`. On Linux: `/tmp, /var/tmp, or /usr/tmp`
4. Git is trying to maintain a state along with ClearML. If these become out of sync it can cause problems and leave you in a state that you will have to recreate the dataset. This can easily be done by `git rm Datasetfolder` in the git repo, `git commit` those deletions, push it, delete the dataset in ClearML and then recreate. Obviously make sure you have a backup of the dataset. This will be improved as more testing and validation is done to catch use cases.
