---
title: OpenAI Gym environment for Franka Emika Panda robot
subtitle: Introducing panda-gym, new RL environments.

# Summary for listings and search engines
summary: Introducing panda-gym, new RL environments.

# Link this post with a project
projects: []

# Date published
date: "2021-02-13T00:00:00Z"

# Date updated
lastmod: "2021-11-08T00:00:00Z"

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption: ''
  focal_point: ""
  placement: 2
  preview_only: false

authors:
- admin

tags:
  - PyBullet
  - OpenAI Gym
  - Robotics
  - Reinforcement Learning
  - grasp

categories:
- Project
---

**Some information contained in this article is not up to date. For up-to-date information, see the technical report available on arXiv.**


## Short intro

It is no longer necessary to explain the genesis of reinforcement learning and the great applications it has. There are many school cases and the learning algorithms are more and more interesting.
I find them particularly fascinating when they can be explained with very simple ideas.

One of the basic environments proposed by OpenAI to train reinforcement learning algorithms are the environments of the [Fetch robot](https://openai.com/blog/ingredients-for-robotics-research/).

This Fetch environments are an excellent starting point to test RL algorithms.
But, you may have noticed that these environments are based on [MuJoCo](http://www.mujoco.org): a very powerful physics engine, but not free.

In this work, I tried to reproduce as faithfully as possible these environments based on a **free** and **open-source** physics engine: [Bullet](https://pybullet.org/wordpress/).
I also chose to use another robot arm, with more joints, and a different gripper: the **Franka Emika Panda robot**.
Source code is available on [GitHub](https://github.com/qgallouedec/panda-gym). 
In the following, I will present you the result of this work, and show you some examples of use.

## Introducing `panda-gym` environments

I am pleased to present 4 new reinforcement learning environments, based on the control in simulation of the Franka Emika Panda robot. These environments, based on the bullet physics engine, try to reproduce as closely as possible the Fetch environments based on MuJoCo.
Here is the list of included environments:

{{< gallery album="panda-gym-v0" >}}

These four environments are `gym.GoalEnv`. This allows the use of learning methods based on the manipulation of `acheived goal` (such as HER, see below).

The action space has four coordinates. The first three are the cartesian target position of the end-effector. The last coordinate is the opening of the gripper fingers. In `PandaReach-v0`, `PandaPush-v0` and `PandaSlide-v0` environments, the fingers are constrained and cannot open. The last coordinate of the action space remains present for a consistency concern.

## Installation and usage

Using `pip` you can install `panda-gym`by running

```bash
pip install panda-gym
```

Then, in a python script, for the `PandaReach-v0` environment:

```python
import gym
import panda_gym

env = gym.make('PandaReach-v0', render=True)

obs = env.reset()
done = False
while not done:
    action = env.action_space.sample() # random action
    obs, reward, done, info = env.step(action)

env.close()
```

For those of you who are used to the OpenAI gym environments, you will notice that the rendering functionality is not handled in the usual way. It is passed as an argument to the `make` function. So you don't need to call the `render()` function.

## Train `panda_gym` with Hindsight Experience Replay

[Hindsight Experience Replay](https://arxiv.org/abs/1707.01495) (HER) was introduced in 2017 by Andrychowicz et al.. The key idea of HER is to see a fail as a success, but with another goal.
It is a method that has shown very promising results in robotic environments. Since the space of action and the space of observation are identical, the results should be very similar.

{{< chart data="chart" >}}

In the early epochs, the Panda seems to learn better than the Fetch. Then, The Panda seems to reach a maximum success rate of about 60%, while the Fetch keeps on learning.

How can these differences in learning performance be explained? Several explanations seem credible. 

- First, since the simulation engine has changed (MujoCo to PyBullet), it is likely that some default parameters make the task more complex: the friction model, the default constants, etc. Future work will have to remove this unknown.
- Second, the robot arm has also changed and therefore the contact surface of the fingers. It is important to note that on the Panda robot simulator, the visually represented contact surface is not the collision surface (the one useful for grasping). For the Fetch robot, the gripping surface is 5.19 cm2 and for the Panda robot it is 5.56 cm2. Thus the Panda robot has a larger gripping surface than the Fetch robot (in simulation only). Nevertheless, I recently noticed that the collision surfaces corresponding to the Panda
robot’s fingers were not strictly parallel. This could reduce the contact area to two contact segments and greatly complicate the gripping task.


## Known limitations

This tool is very satisfying for testing deep reinforcement learning algorithms. However, some points can be limiting. Here is a list of them.

1. It is not possible to easily deploy the policy learned in simulation, for several reasons:
   - The Franka Emika Panda robot works with the [`libranka`](https://frankaemika.github.io/docs/libfranka.html) library. This library allows a control of the joints but not of the cartesian position of the end-effector. To solve this issue, it is possible to use the [`franka_ros`](https://frankaemika.github.io/docs/franka_ros.html) overlay and [MoveIt!](http://docs.ros.org/en/kinetic/api/moveit_tutorials/html/) to control the cartesian position of the end-effector. The limitation remains in the gripper control: it can only be controlled by high level actions such as `grasp` and `move`. Additional work is required to allow the deployment of a policy learned in simulation.

   - The simulation is stopped when the action is inferred. It is not possible to stop real time. Several tricks are possible, future work will explore them.

2. The simulation is not completely realistic.
One difference already mentioned concerns the shape of the gripper. The gripper collision mesh is shown in the following figure.

![Gripper collision shape](collision_gripper.jpg "Gripper collision shape")

In reality, the surface is much smaller, and does not have this shape. Then, the dynamic behavior is surely not faithful to reality since the masses and inertia matrices taken from from `pybullet` are not correct. Nevertheless, I do not propose any quantification of this behavioral deviation.

## Additionnal material

For more information on this subject, please consult:
- [detailed report](../files/TFErapport.pdf)
- [additionnal results](../files/recent%20results.pdf)
- [Panda-gym GitHub repository](https://github.com/qgallouedec/panda-gym)
- [Code used for training](https://github.com/qgallouedec/drl-grasping)








