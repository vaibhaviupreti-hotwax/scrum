# OFBiz 24.09 Setup Walkthrough

The setup for Apache OFBiz 24.09 has been successfully completed.

## Steps Taken

1. **Repository Preparation**:
   - Switched to the `release24.09` branch.
   - Initialized the Gradle wrapper by running `./gradle/init-gradle-wrapper.sh`. This step downloaded the necessary `gradle-wrapper.jar`.

2. **Build and Data Load**:
   - Executed `./gradlew cleanAll loadAll`.
   - This command compiled the Java and Groovy source code.
   - It also initialized the database and loaded both seed and demo data.
   - The process completed successfully in approximately 5 minutes.

## Verification Results

- **Build Status**: `BUILD SUCCESSFUL`
- **Actionable Tasks**: 30 actionable tasks (25 executed, 5 up-to-date).

## How to Run OFBiz

To start the OFBiz server, run the following command from the `ofbiz-framework` directory:

```bash
./gradlew ofbiz
```

Once started, you can access the OFBiz suite at:
- **HTTPS**: `https://localhost:8443/partymgr` (or other components)
- **Default Username**: `admin`
- **Default Password**: `ofbiz`
