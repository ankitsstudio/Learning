# VS code setup and deployment setup and running test locally

## Install Python version
### 1. install python version 3.10.11 (is suitable for azureml-core and other requirements)

### 2. create your virtual environment for the dependencies to install, using below command

* **Run Command**:

  ```bash
  python -m venv .venv
  ```

  this will create a virtual environment for your python project, we need to activate the virtual environment

* **Run Command**:

  ```bash
  ./venv/Script/Activate
  ```
---

### 3. Install all the dependencies for the project, add a requirements.txt file in the AML-Runners/tests/scenerio/agents-era with below dependencies
   ```bash
pytest
azure-ai-ml
azure-ai-agents
azure-ai-projects
azure-identity
azure-mgmt-network
azure-mgmt-storage
azure-mgmt-search
azure-mgmt-applicationinsights
azure-storage-blob
azure-search-documents
azureml-core
azureml-sdk
setuptools
jsonref
azure-cli
jsmin==3.0.1
   ```

   * **Run Command**:

  ```bash
  pip install -r .\requirements.txt
  ```


## 🧪 How to Run Tests Locally

To test locally, first navigate to the `tests\scenarios\agents-era` directory.

### 1. `private_1rp_basic_tests.py`

* **Modify Resource Config**:
  Ensure that the `private_basic_agent_config` in `constants_1rp.py` is configured with the resources for the target region.

* **Run Command**:

  ```bash
  pytest private_1rp_basic_tests.py -s -v --region REGION_TO_BE_TESTED | Tee-Object -FilePath "log.txt"
  ```

---

### 2. `private_byo_1rp_standard_tests.py`

* **Modify Resource Config**:
  Ensure that the `private_standard_agent_config` in `constants_1rp.py` is configured with the resources for the target region.

* **Run Command**:

  ```bash
  pytest private_byo_1rp_standard_tests.py -s -v --region REGION_TO_BE_TESTED | Tee-Object -FilePath "log.txt"
  ```

---

### 3.1 `public_1rp_basic_tests.py` – Using Custom Resource

* **Modify Resource Config**:

  1. In `conftest.py`, make the `is_local()` function return `True`.
  2. Ensure `local_basic_agent_config` in `constants_1rp.py` has the correct resources for the target region.

* **Run Command**:

  ```bash
  pytest public_1rp_basic_tests.py -s -v --region REGION_TO_BE_TESTED | Tee-Object -FilePath "log.txt"
  ```

---

### 3.2 `public_1rp_basic_tests.py` – Using Today's Pipeline Resource

* **Run Command**:

  ```bash
  pytest public_1rp_basic_tests.py -s -v --region REGION_TO_BE_TESTED | Tee-Object -FilePath "log.txt"
  ```

---

### 4.1 `public_1rp_standard_tests.py` – Using Custom Resource

* **Modify Resource Config**:

  1. In `conftest.py`, make the `is_local()` function return `True`.
  2. Ensure `local_standard_agent_config` in `constants_1rp.py` has the correct resources for the target region.

* **Run Command**:

  ```bash
  pytest public_1rp_standard_tests.py -s -v --region REGION_TO_BE_TESTED | Tee-Object -FilePath "log.txt"
  ```

---

### 4.2 `public_1rp_standard_tests.py` – Using Today's Pipeline Resource

* **Run Command**:

  ```bash
  pytest public_1rp_standard_tests.py -s -v --region REGION_TO_BE_TESTED | Tee-Object -FilePath "log.txt"
  ```

---

### 5. `public_byo_tests.py`

* **Modify Resource Config**:
  Ensure `agents_config` in `constants_.py` contains the correct resources for the target region.

* **Run Command**:

  ```bash
  pytest public_byo_tests.py -s -v --region REGION_TO_BE_TESTED | Tee-Object -FilePath "log.txt"
  ```

---

### 6. `public_deployment_tests.py` ⚠️

* **Important Notes**:

  * Operates **only** on **1RP Public** resources
  * Cleans up **yesterday's and older** resource groups and all contained resources
  * Creates **tomorrow’s** resource group and deploys:

    * 1RP Standard resources
    * 1RP Basic resources

* **Run Command**:

  ```bash
  pytest public_deployment_tests.py -s -v --region REGION_TO_BE_TESTED | Tee-Object -FilePath "log.txt"
  ```

* **Caution**: Use with care, as this test is destructive and modifies shared environments.

---

### 📋 Note on Output Redirection

The `| Tee-Object -FilePath "log.txt"` part of the command is used to **both display the output in the console** and **save it to a file** for debugging purposes.
If you only want to see the output in the console and don't need to log it to a file, you can **remove this part** of the command:

```bash
pytest your_test_file.py -s -v --region REGION_TO_BE_TESTED
```

---

### 🔬 Running a Specific Test Case

The above examples demonstrate how to run tests **at the file level**.
If you want to run a **specific test case** within a file, replace the filename in the command with:

```
<filename>::<test_case_function_name>
```

#### ✅ Example:

To run only the `test_private_1rp_list_agents` case in `private_1rp_basic_tests.py`, use:

```bash
pytest private_1rp_basic_tests.py::test_private_1rp_list_agents -s -v --region REGION_TO_BE_TESTED | Tee-Object -FilePath "log.txt"
```

This helps to narrow down the scope when debugging or validating a single scenario.

