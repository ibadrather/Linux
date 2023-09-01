## Systemd Unit Files

At the heart of **systemd's** management capabilities are unit files. These are configuration files that define and control resources that **systemd** knows how to manage. These resources are referred to as "units" and can represent services, mount points, devices, etc.

### 1. Types of Units:

While there are many types of units, some of the most commonly used ones include:

**.service**: Represents a process or group of processes. This is probably the most familiar type of unit for many Linux users, as it's used to start and manage background services.

**.mount**: Represents a filesystem mount point.

**.device**: Represents a device in the system, and it's dynamically created by **systemd** from the **udev** device manager.

**.socket**: Used for IPC (inter-process communication). It allows for starting services on-demand when they're accessed.

**.target**: A grouping mechanism that allows for batch management of units, representing system states like **multi-user.target** or **graphical.target**.

**.timer**: Represents a timer, triggering activation of other units based on time.

... and there are others, such as **.path**, **.slice**, **.swap**, etc.

#### multi-user.target

In systemd, a target is a grouping mechanism that allows for batch management of services and units. Targets are used to implement systemd's concept of system states; each target collects a batch of services and other units to be managed together.

**multi-user.target** is one of the core system targets within systemd, and it serves as a named group of services and other units that are essential for a multi-user system setup, typically for systems that are used by multiple users but don't necessarily have a graphical interface.

Here's a more detailed breakdown:

- **Purpose**: The** multi-user.target** is intended for multi-user systems. It's analogous to the traditional runlevel 3 in SysV init systems, where the system boots into multi-user mode with networking but without a graphical user interface.

- **What it Includes**: It generally includes services like networking, logging, cron, and other essential services that are required to operate a system that might be used by multiple users.

- **Dependencies**: By default, in most systems, **multi-user.target** is a dependency of **graphical.target**, meaning that if you're booting into a graphical environment, the services required by **multi-user.target** are started first.

- **Usage**: If you want to set your system to boot into a multi-user, command-line environment by default (without a GUI), you can set **multi-user.target** as the default target:

```bash
sudo systemctl set-default multi-user.target
```

**Checking Status**: To see the status of this target and which services it pulls in, you can use:

```bash
systemctl list-dependencies multi-user.target
```

In essence, **multi-user.target** represents a certain operational state of the system. By selecting this target, you're essentially saying, "I want my system to have all the capabilities needed for multiple users to work from the command line." If you're familiar with the concept of runlevels from SysV or Upstart init systems, **multi-user.target** is systemd's more flexible and powerful replacement for some of those concepts.

### 2. Structure of a Unit File:

Unit files have a structure similar to Windows INI files, consisting of sections marked by **[...]** and key-value pairs within those sections.

For example, a simple service unit might look like:

```ini
[Unit]
Description=My Custom Service

[Service]
ExecStart=/path/to/my/command
Restart=always

[Install]
WantedBy=multi-user.target
```

- **[Unit]** section provides a general description and can also define dependencies with other units.

- **[Service]** section provides directives specific to services, like the command to execute when the service starts.

- **[Install]** section defines the behavior if the unit is enabled or disabled.

### 3. Dependencies:

Dependencies are a crucial aspect of **systemd** units. They define the order and conditions under which units are activated. Some common dependency directives include:

- **Requires**: If the current unit gets activated, the units listed will also be activated.

- **Wants**: Like Requires, but its failure is considered non-critical.

- **After** or **Before**: Defines the order in which units are started.

### 4. Where Unit Files are Located:

Systemd service files (and other unit files) can be found in several locations on a typical Linux system, based on their purpose and the entity that installed them:

- **System Packages / Default Units**:

    - **/lib/systemd/system/** (on Debian, Ubuntu, and related distributions)
    - **/usr/lib/systemd/system/** (on Red Hat, Fedora, and related distributions)

    This directory contains unit files provided by the system packages. It's populated by default services that come with the OS or services that are installed through a package manager like **apt**, **yum**, or **dnf**.

- **Local Customization / Overrides**:

    - **/etc/systemd/system/**

    This is where system administrators typically place their custom service files or override the default ones. If a unit with the same name exists in both /etc/systemd/system/ and /lib/systemd/system/, the one in /etc/ takes precedence. This directory can also contain additional configuration in the form of drop-in files located in subdirectories named <unit>.d/.

- **User Services**:

    **~/.config/systemd/user/**
    Individual users can have their own systemd user services, and they go in this directory in the user's home directory. These units are managed by the user's instance of systemd, not the system's main systemd process.

- **Run-time Units**:

    **/run/systemd/system/**
    
    This directory is for units that are generated on-the-fly during system operation. These aren't persistent across reboots.

If you've created a custom service or you're troubleshooting a service, it's likely in the **/etc/systemd/system/** directory. If you're examining services provided by installed packages, look in **/lib/systemd/system/** or **/usr/lib/systemd/system/**, depending on your distribution.

### 5. Managing Unit Files:

The primary command to manage **systemd** units is **systemctl**. Examples include:

- **systemctl start myservice.service**: Start a service.

- **systemctl enable myservice.service**: Enable a service to start at boot.

- **systemctl status myservice.service**: Check the status of a service.

In essence, unit files allow **systemd** to understand and manage various system components and resources in a standardized manner, ensuring smooth boot processes, efficient resource management, and dependency resolution.

### Example of a systemd unit which runs a python script

Let's say you have a Python script named myscript.py that you'd like to run as a systemd service. Here's how you can do that:

#### 1. Python Script
Make sure your Python script has the correct shebang line at the top, so it can be executed directly. The shebang line tells the system how to execute the file.

For example, if you're using Python 3:

```python
#!/usr/bin/env python3

# Your Python code here...
print("Hello from my Python script!")
```

Ensure the script has execute permissions:

```bash
chmod +x /path/to/myscript.py
```

#### 2. Systemd Service Unit
Now, create a systemd service unit file for the script:

/etc/systemd/system/myscript.service:

```ini
[Unit]
Description=My Python Script Service
After=network.target

[Service]
ExecStart=/path/to/myscript.py
Restart=always
User=username
Group=groupname
Environment=PATH=/usr/bin:/usr/local/bin
WorkingDirectory=/path/to/script/directory

[Install]
WantedBy=multi-user.target
```

Here's a brief explanation of the configuration:

- **Description**: A brief description of the service.

- **After**: Specifies that the service should start after the network is up.

- **ExecStart**: The command to start the service (your Python script).

- **Restart**: Tells systemd to restart the service whenever it exits. You can adjust this as needed; for instance, **on-failure** will only restart it if the script exits with an error.

- **User** and **Group**: Specify under which user and group the script should run.

- **Environment**: Sets the PATH variable to ensure the environment knows where to find Python.

- **WorkingDirectory**: Specifies the working directory for the script (can be useful if your script reads/writes files relative to its location).

- **WantedBy**: Specifies that the service is wanted by multi-user.target, meaning it'll start during the boot process once multi-user mode is reached.

#### 3. Start and Enable the Service
After creating the service file:

1. Reload the systemd manager configuration:

```bash
sudo systemctl daemon-reload
```

2. Start the service:

```bash
sudo systemctl start myscript.service
```

3. If you want the service to start automatically on boot:

```bash
sudo systemctl enable myscript.service
```

4. Check the status of your service:

```bash
sudo systemctl status myscript.service
```

Now, your Python script will run as a systemd service! Adjust the service configuration as needed to tailor it to your specific requirements.
