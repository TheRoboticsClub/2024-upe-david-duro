---
title: JdRobot Internship Blog
---

# <img src="imgs/logo.png" alt="JdRobot" style="height: 100px;">

## Welcome to my blog!

Here I will write everything I learn during my internship at JdRobot.

## Index

- [Welcome to my blog!](#welcome-to-my-blog)
- [Building GitHub Pages](#building-github-pages)
- [Launching RoboticsAcademy Docker](#launching-roboticsacademy-docker)
- [Contact](#contact)

## Building GitHub Pages

GitHub Pages is a simple way to host static websites directly from a GitHub repository. Follow these steps to set up your own GitHub Pages site:

1. **Create a Repository**:
   Go to GitHub and create a new repository. If you want your site to be available at `https://your-username.github.io`, name the repository `your-username.github.io`.

2. **Add Your Site Content**:
   Upload your index file (HTML, CSS, JavaScript). I'm using Markdown for simplicity.

3. **Configure GitHub Pages**:
   Go to the repository settings, scroll down to the "Pages" section, and select the branch you want to use for your site. If your files are in a specific folder (like `docs`), select that folder.

4. **Access Your Site**:
   After a few minutes, your site will be live at `https://your-username.github.io/repository-name` (or `https://your-username.github.io` if you named the repository after your username).

For more detailed instructions, visit the [GitHub Pages documentation](https://docs.github.com/en/pages).

## Launching RoboticsAcademy Docker

RoboticsAcademy allows you to launch the docker locally without the need for an internet connection and perform a wide variety of robotics exercises using the open-source Gazebo simulator. The user simply creates a Python program, which will interact with ROS underneath.

### Performance Metrics

Exercises from Service Robotics.

SO: Ubuntu 22.04.4 LTS, RAM: 14 GB, CPU: AMD Ryzen 5 7535HS (6 cores):

| Task                     | Gazebo’s RTF | Gazebo’s FPS | % CPU Usage |
|--------------------------|--------------|-------------|------------|
| Autoparking              | 0.88         | 5           | 95%        |
| Localized Vacuum Cleaner | -         | -          | -        |
| Rescue People            | 0.99         | 18          | 99%        |
| Amazon Warehouse         | 0.97         | 5           | 99%        |



### Conclusions

All the exercises work correctly, as in the previous version, except the self-localizing vacuum cleaner exercise, I have seen that the map path has changed, but even modifying it, it does not work correctly. I have also checked that they work importing the modules in any of these ways:

Last version:
```bash
from GUI import GUI
from HAL import HAL
```

New version:
```bash
import GUI
import HAL
```

## Contact

You can contact me through [davidduro2002@gmail.com](mailto:davidduro2002@gmail.com).

---

&copy; 2024 JdRobot Internship Blog. All rights reserved.
