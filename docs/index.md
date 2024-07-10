---
title: JdRobot Internship Blog
---

# <img src="imgs/logo.png" alt="JdRobot" style="height: 100px;">

## Welcome to my JdRobot Internship Blog!

Here I will write everything I learn during my internship at JdRobot.

## Index

- [Welcome to my JdRobot Internship Blog!](#welcome-to-my-jdrobot-internship-blog)
- [Building GitHub Pages](#building-github-pages)
- [Launching RoboticsAcademy Docker](#launching-roboticsacademy-docker)
- [Developer Enviroment](#developer-enviroment)
- [Solving First Issue](#solving-first-issue)
- [JdRobot Division](#jdrobot-division)
- [issue-2640](#issue-2640)
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

#### No GPU Acceleration
SO: Ubuntu 22.04.4 LTS, RAM: 16 GB, CPU: AMD Ryzen 5 7535HS (6 cores):

| Task                     | Gazebo’s RTF | Gazebo’s FPS | % CPU Usage |
|--------------------------|--------------|-------------|------------|
| Autoparking              | 0.88         | 5           | 95%        |
| Localized Vacuum Cleaner | -         | -          | -        |
| Rescue People            | 0.99         | 18          | 99%        |
| Amazon Warehouse         | 0.97         | 5           | 99%        |


### Conclusions

All the exercises work correctly, as in the previous version, except the self-localizing vacuum cleaner exercise, I have seen that the map path has changed, but even modifying it, it does not work correctly (I finally realized that the getMap() function has some problems). I have also checked that they work importing the modules in any of these ways:

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

## Developer Enviroment

First of all, we have to install all the requierements. You have detailed information here: [Instructions for developers](https://github.com/JdeRobot/RoboticsAcademy/blob/humble-devel/docs/InstructionsForDevelopers.md)

### Starting Problems

I have encountered several errors when launching the script that creates the developer environment.

So I had to reinstall nvm this way:
```bash
nvm use 16
nvm install-latest-npm
```

And then reinstall yarn:
```bash
npm uninstall -g yarn
npm install -g yarn
```

### How to launch developer enviroment

```bash
cd <dir>/RoboticsAcademy
```
First we must launch the frontend like this:
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
nvm install 16
nvm use 16
cd react_frontend/ && yarn install && yarn run dev
```

Finally we launch the script (in other terminal):
```bash
sudo sh scripts/develop_academy.sh
```


Using `Ctrl+c` will stop them.

## Solving first issue

Once we have the frontend launched and the script running, we can make changes and see them in real time.

The exercises proposed by JdeRoboticsAcademy feature two main classes, GUI and HAL, both of which are essential for the functioning of all exercises. HAL is based on the robot's hardware, its sensory functions. GUI allows us to modify frontend elements (maps, images), and to receive resource information from it.

When I was testing my exercises made in Service Robotics there was one that did not work correctly because a function was giving problems. For this reason, my goal has been to test modifying and trying to fix this function.

#### getMap()

Honestly, I didn't find the reason for the error in the function, it simply imported a library that for some reason didn't work and returned an empty image. I simply used another library and instead of returning the image with 4 channels, I returned it in RGB with some debugging elements:

```python
# humble-devel branch
def getMap(self, url):        
   return plt.imread(url)
```

```python
# my branch
from PIL import Image

def getMap(self, url):
   try:
   # Open with PIL
      with Image.open(url) as img:
            img = img.convert("RGB")
            img_array = np.array(img)
      return img_array
   except Exception as e:
      print(f"Error reading image from {url}: {e}")
      return None
```


#### showNumpy()

After checking that the image was loaded correctly I got another error from the showNumpy() function.

The problem at first seemed straightforward to me as it simply appeared that a list called `shared_image` was not declared.

```python
# humble-devel branch
def showNumpy(self, image):
   self.shared_image.add(self.process_colors(image))
```

```python
# my branch
from shared.image import SharedImage

def __init__(self, host):
   # Code....
   self.shared_image = SharedImage("guiimage")

# Code....

def showNumpy(self, image):
   self.shared_image.add(self.process_colors(image))
```

At this point the bug was apparently fixed, the problem came with the error in the following function.

#### process_colors()
The problem here was not so simple, besides, it was my mistake not to read the documentation on how it worked. Once I understood how it works (in my opinion it can be changed for simplicity, but all the people who have used it as it is implemented would stop working for them).

Here the error message:
`IndexError: boolean index did not match indexed array along dimension 2; dimension is 3 but corresponding boolean dimension is 1`

It took me quite a while to realise what was going on, until I debugged the size of the arrays and mask used.

The original comparation:
```python
# humble-devel branch
mask = image < 128
colored_image[mask] = image[mask][:, None] * 2
```

```python
# my branch
#### NOT USED ----- EXPLAINED ABOVE -----
def process_colors(self, image):
   if image.ndim != 3 or image.shape[2] != 1:
      print(f"The input image must be a 3D array with a single layer (height, width, 1).")
      return None

   # Grayscale image without last dim
   grayscale_image = image[:, :, 0]

   # Color exit image
   colored_image = np.zeros((grayscale_image.shape[0], grayscale_image.shape[1], 3), dtype=np.uint8)

   # Color lookup table
   color_table = {
      128: red,
      129: orange,
      130: yellow,
      131: green,
      132: blue,
      133: indigo,
      134: violet
   }

   # Grayscale for values < 128
   gray_mask = grayscale_image < 128
   colored_image[gray_mask] = grayscale_image[gray_mask][:, None] * 2

   for value, color in color_table.items():
      color_mask = (grayscale_image == value)
      colored_image[color_mask] = color

   return colored_image
```

The problem was in the comparison between an image with 3 channels and a greyscale image with only 1 channel, I solved it by adding a new matrix.

### New Problems

Once I was able to solve all the bugs, I got the code to run without any problems (my code does not work correctly as the speed controller was tested in ROS1). But the pre-navigation part should still work correctly, which I couldn't check as the map was not being updated. At first I thought I had done something wrong and showNumpy() was still having problems. So I decided to take a look at the ROS1 gui.py class and compare them. This is when I realised that the part that updates and publishes the images is still not implemented. Which is something that is out of my knowledge for now.

### Change of format

We must modify our array to a format that json understands.

### New window

To show the new map created by the user we must create a new window next to the original map, which will be updated when the user calls the showNumpy() function. The problem is that for now I don't know how to create the new window and I can't update the already created window either.

### Actualization

A few days later, thanks to a collaborator, we were able to copy the functionality of displaying the new ros1 window and fix the functions I mentioned above. The process colours function was correct, it didn't work for me because during my work with that practice the function was different.

By simply fixing the getmap() function and adding the new window, we fixed the exercise. (issue-2637)

## JdRobot Division

### Robots Academy

Repository with the exercises and frontend, here are the classes and the specific functions of each exercise.

### RoboticsApplicationManager

Repository for communications between `browser` - `manager` - `user`.

### RoboticsInfrastructure

Repository with robot models, gazebo maps and launchers.

## issue-2640

GUI was divided into two classes, which was not efficient as this could actually be simplified. We simplified the functions and created a single global class called ThreadingGUI. Once this was done, we realised that acks are never checked. We also solved the problem.

## Contact

<a href="https://linkedin.com/in/david-duro-aragon%c3%a9s-32a0a4272" target="blank">
   <img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/linked-in-alt.svg" alt="LinkedIn" height="30" width="40" />
</a>
<a href="https://instagram.com/__d_u_r_o__" target="blank">
   <img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/instagram.svg" alt="Instagram" height="30" width="40" />
</a>
<a href="https://github.com/dduro2020" target="blank">
   <img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/github.svg" alt="Github" height="30" width="40" />
</a>
<a href="mailto:davidduro2002@gmail.com" target="blank">
   <img align="center" src="https://logowik.com/content/uploads/images/gmail-new-icon5198.jpg" alt="Mail" height="30" width="40" />
</a>


---

&copy; 2024 JdRobot Internship Blog. All rights reserved.
