# Pytest
1. Pytest is a testing framewrok for python language.
2. It is used to write unit tests, integration tests and end-to-end tests.
3. It comes with some powerful features like Fixtures, assert rewriting, plugins and parameterization.

## Fixtures
In pytest, a fixture is function that sets up some state or provides resources that your tests can use

- They are mainly used for setup and teardownn logic.
- Example: creating a database connection, initialization test data, preparing a temporary directory, or setting up environment variables.
- If we define fixture inside a file named conftest.py, pytest will automacilly discover them
- Any test file inside the same folder (and subfolder) can use those fixtures without importing them.

### Scope of fixtures
Fixture can have different scopes, which determine how long they live and where they can be accessed. Scope controls how a fixture lives and when it gets created/destroyed during the test execution. Instead of recreating a fixture every time, pytest can reuse the same fixture across multiple tests depending on the scope of we choose.

**Folder structure**
``` csharp
tests/
│
├── conftest.py   # <-- define fixtures here
├── test_users.py
├── test_orders.py
```

**Conftest.py**
``` python
import pytest

@pytest.fixture
def api_client():
    return "Fake API Client"
```

- So, if we have the `api_client` fixture defined within conftest file, both `test_users.py` and `test_orders.py` file can use it without importing it because pytest makes it **available to all tests in a folder**.

**Available Scopes in pytest**
1. **Function (default)**
- fixture is created **fresh for every test function**.
- Best for cases where each test needs a clean, isolated environment.
- Useful for: unit tests where we don't want shared state.

2. **Class**
- Fixture is created **once per test class**.
- All test methods inside that class share the same fixture instance.
- Useful for: tests grouped in a class that can share a setup.

3. **Module**
- Fixture is created **once per test file (module)**.
- Shared across all test functions and classes in that file.
- Useful for: expensive resources you want to share across all tests in the same file (e.g., starting a mock server).

4. **Session**
- Fixture is created only **once for the entire test run** (regardless of number of files).
- Useful for: global resources like a database connection, external service, or test data that should persist.

**Example**
```python
@pytest.fixture(scope='session')
def shared_agent_1rp_basic_data_plane(region, is_local):
    print_log(f"start shared_agent_1rp_basic_data_plane: region = {region};")
    return get_agent_workspace(
        region=region,
        is_local=is_local,
        type_prefix=agents_constants_1rp.default_basic_prefix,
        local_resources=agents_constants_1rp.local_basic_agent_test_resources,
        check_region_fn=avaliable_1RP_basic_region,
        model_config=agents_constants_1rp.basic_agents_test_resources
    )
```
- This fixture has `scope='session'`, so it is set up once for the entire pytest run.
- All tests that use `shared_agent_1rp_basic_data_plane` will share the same instance.

**How does this behave for tests in** `public_1rp_basic_tests.py`? For example:
``` python
def test_1rp_creation_openapi(region, shared_agent_1rp_basic_data_plane):
    base.test_1rp_creation_openapi(region, shared_agent_1rp_basic_data_plane)
```
- When pytest runs all tests in `public_1rp_basic_tests.py`, it will call the fixture only once, and reuse the same workspace for every test that requests it.
- This is efficient for expensive setup (like creating a workspace or connecting to a remote service) that doesn't need to be repeated for every test.

### Fixture concepts
1. **autouse = True**
- When we set `autouse=True` in a fixture, pytest will automatically apply that fixture to all tests in its scope (function, class, module, or session), even if the test does not explicitly request it.
- **Use case**: For setup/teardown that should always run, like logging, environment setup, or cleaning resources.

*Example:*
``` python
@pytest.fixture(scope="session", autouse=True)
def shared_login(is_local):
    if is_local is True:
        logger.info("Local test, using user's identity")
        commands.run_cli_command(["az", "login"], do_not_print=False)

        yield
        return
    try:
        with string_helpers.PrintAtStartAndEnd("Notebooks CLI system session startup"):
            original_directory = os.path.abspath(".")
            output_directory = shared_resources.get_output_directory()
            os.chdir(output_directory)

            account.login_as_runners_sp()
            account.set_active_account(
                cli_to_use=constants.AZURE_CLI_INFO,
                subscription_id=constants.SUBSCRIPTION)
        yield
    finally:
        with string_helpers.PrintAtStartAndEnd("Project system session teardown"):
            os.chdir(original_directory)
```
- All tests will have logging shared set up, without needing to mention `shared_login` in the test function.

2. **Fixture Dependencies (fixtures calling fixtures)**
- Fixtures can depend on other fixtures by listing them as arguments. Pytest will resolve and inject those dependencies automatically.
- This allows us to build complex setups from simple, reusable pieces.

*Example:*
``` python 
@pytest.fixture(scope="session", autouse=True)
def is_local():
    # for local test, set corresponding environment variables
    return not (runner_region_env_var in os.environ and subscription_env_var in os.environ)

@pytest.fixture(scope="session")
def region(request):
    region_str = request.config.getoption("--region")
    if not region_str:
        region_str = constants.default_location
    print_log(f"Region: {region_str}")
    if avaliable_region(region_str) is False:
        print_log(f"Agent Test is not available in {region_str}")
        pytest.skip(f"Agent Test is not available in {region_str}")
    return region_str

@pytest.fixture(scope='session')
def shared_agent_1rp_basic_data_plane(region, is_local):
    print_log(f"start shared_agent_1rp_basic_data_plane: region = {region};")
    return get_agent_workspace(
        region=region,
        is_local=is_local,
        type_prefix=agents_constants_1rp.default_basic_prefix,
        local_resources=agents_constants_1rp.local_basic_agent_test_resources,
        check_region_fn=avaliable_1RP_basic_region,
        model_config=agents_constants_1rp.basic_agents_test_resources
    )
```
- Here, `shared_agent_1rp_basic_data_plane` depends on `region` and `is_local`, so pytest will call `region` and `is_local` first and pass its result to `shared_agent_1rp_basic_data_plane`.

3. **Parameterized Fixtures (`params=[...]`)**
- We can use the `params` argument to run a fixture multiple times with different values. Each test using the fixture will run once for each parameter.
- Useful for testing with multiple configurations or data sets.

*Example:*
``` python 
@pytest.fixture(params=[1, 2, 3])
def number(request):
    return request.param

def test_is_even(number):
    assert number % 2 == 0
```
This test will run three times: once with `number=1`, `number=2`, and `number=3`.

4. **Using fixtures with markers**
- Markers are used to add metadata to tests (e.g., `@pytest.mark.slow`, `@pytest.mark.parametrize`, `@pytest.mark.owner`). We can use markers to control which tests use which fixtures, or to parametrize tests.
- We can also use custom markers to group tests or apply conditions.

*Example:*
``` python 
@pytest.mark.parametrize("input,expected", [(1,2), (2,3)])
def test_increment(input, expected):
    assert input + 1 == expected

@pytest.mark.slow
def test_long_running(shared_agent_1rp_basic_data_plane):
    # This test is marked as slow and uses a fixture
    ...
```
- `@pytest.mark.parametrize` runs the test with different inputs.
- `@pytest.mark.slow` can be used to select or skip slow tests.

---

## Decorators & Wrapper

* **Our Code:**
```python 
def setup_v2_agent_client(region, shared_agent_v2_data_plane):
    """Setup V2 agent client
    
    Args:
        region: Region name
        shared_agent_v2_data_plane: Workspace configuration
        
    Returns:
        V2AgentClient instance
    """
    print_log(f"V2: Setting up agent client for region {region}")
    
    credential = DefaultAzureCredential()
    agent_client = V2AgentClient(
        endpoint=shared_agent_v2_data_plane.endpoint,
        credential=credential,
        subscription_id=shared_agent_v2_data_plane.subscription_id,
        resource_group=shared_agent_v2_data_plane.resource_group,
        workspace_name=shared_agent_v2_data_plane.project_workspace_name
    )
    
    print_log(f"V2: Agent client created successfully")
    return agent_client


def cleanup_v2_resources(agent_client, disposable_list):
    """Cleanup V2 resources
    
    Args:
        agent_client: V2AgentClient instance
        disposable_list: List of resources to cleanup
    """
    print_log(f"V2: Starting cleanup of {len(disposable_list)} resources")
    
    for resource_type, resource_data in reversed(disposable_list):
        try:
            if resource_type == agents_constants_v2.V2CleanupResourceType.AGENT_VERSION:
                agent_name, version = resource_data
                print_log(f"V2: Cleaning up agent version: {agent_name}/{version}")
                agent_client.delete_agent_version(agent_name, version)
            elif resource_type == agents_constants_v2.V2CleanupResourceType.AGENT:
                agent_name = resource_data
                print_log(f"V2: Cleaning up agent: {agent_name}")
                agent_client.delete_agent(agent_name)
            else:
                print_log(f"V2: Unknown resource type: {resource_type}", "warning")
        except Exception as e:
            print_log(f"V2: Error cleaning up resource: {e}", "error")
    
    print_log("V2: Cleanup completed")


def v2_agent_client_context(func):
    """Context manager decorator for V2 agent client setup and teardown"""
    @wraps(func)
    def wrapper(region, shared_agent_v2_data_plane, *args, **kwargs):
        agent_client = setup_v2_agent_client(region, shared_agent_v2_data_plane)
        disposable_list = []
        try:
            func(region, shared_agent_v2_data_plane, agent_client, disposable_list, *args, **kwargs)
        except Exception as e:
            print_log(f"V2: Error during test execution: {str(e)}", "error")
            raise
        finally:
            cleanup_v2_resources(agent_client, disposable_list)
    return wrapper


# V2 Test Functions

@with_retry
@v2_agent_client_context
def test_v2_create_agent(region, shared_agent_v2_data_plane, agent_client, disposable_list):
    """Test creating an agent version with hosted container
    
    This test:
    1. Creates an agent version with hosted container
    2. Verifies the agent version was created successfully
    """
    testcase = "test_v2_create_agent"
    print_log(f"V2: Test case name: {testcase}, test region - {region}")

    # Create agent version
    agent_name = generate_unique_name("create-test-agent")
    
    # Use a test container image
    test_image = os.environ.get("TEST_AGENT_IMAGE", "mcr.microsoft.com/azureml/aifx/stable-ubuntu2004-cu121-py310-torch230:latest")
    
    print_log(f"V2: Creating agent version with name: {agent_name}")
    agent_version = agent_client.create_agent_version(
        agent_name=agent_name,
        image=test_image,
        cpu="1",
        memory="2Gi",
        description="Test hosted agent for create test"
    )
    
    # Add to cleanup list
    disposable_list.append((
        agents_constants_v2.V2CleanupResourceType.AGENT_VERSION, 
        (agent_name, agent_version['version'])
    ))
    
    print_log(f"V2: Created agent version: {agent_version}")
    
    # Verify agent version was created
    assert agent_version['name'] == agent_name, f"Agent name mismatch: {agent_version['name']} != {agent_name}"
    assert agent_version['version'] is not None, "Agent version is None"
    assert agent_version['object'] == "agent.version", "Object type mismatch"
    
    print_log(f"V2: Test {testcase} passed successfully!")

```

### 1. What is a Decorator in Python?
A **decorator** is a **function that modifies the behavior of another function**, without changing its actual code.
It’s a cleaner, reusable way to handle **setup / teardown / error handling / logging / retrying** logic around a function.

**Example:**
```python
def decorator(func):
    def wrapper():
        print("Before running function")
        func()
        print("After running function")
    return wrapper

@decorator
def say_hello():
    print("Hello!")

say_hello()
```

**Output**:
```sql
Before running function
Hello!
After running function
```

* Here, @decorator wraps the say_hello() function inside another function that adds extra logic.

### 2. What is a Wrapper?
The **wrapper** is the **inner function** inside the decorator — the one that actually runs before and after your original function.

**Example:**
```python
def v2_agent_client_context(func):
    @wraps(func)
    def wrapper(region, shared_agent_v2_data_plane, *args, **kwargs):
        ...
    return wrapper
```
* `v2_agent_client_context` → the **decorator**
* `wrapper(...)` → the `function that wrap`s your test
* `func(...)` → your **original test function**, e.g. `test_v2_create_agent`
  
### 3. Why `@wraps(func)` is used?
`@wraps(func)` (from `functools`) preserves metadata of the original function like its:
* `__name__`
* `__doc__`
* `__module__`

Otherwise, your test function’s name would show up as just `"wrapper"` in pytest reports, which would be confusing.

### 4. In your code — `@v2_agent_client_context`
Here’s how it works in your code:

```python
@with_retry
@v2_agent_client_context
def test_v2_create_agent(...):
```

**Execution Flow:**
When pytest runs this test:
* `@v2_agent_client_context` runs first.
  * It calls `setup_v2_agent_client()` → creates and configures the client.
  * Passes that client and an empty `disposable_list` to the test function.
  * When the test finishes (or fails), it runs `cleanup_v2_resources()` to delete created resources.
  * So, even if the test fails midway, your Azure resources don’t leak.

* `@with_retry` (outer decorator) adds retry logic.
  * It probably reruns the test automatically if it fails (like flaky network or transient error).

So effectively, you’ve built a **context manager-like test decorator** that handles setup, cleanup, and retries **automatically**.