+++
date = '2025-07-16T16:52:57+10:00'
draft = false
image = "/assets/images/atari-breakout-display.jpg.webp"
title = 'Playing Atari with deep reinforcement learning '
+++


{{< sidenote >}}
![atari-breakout](/assets/images/atari-breakout-display.jpg.webp)

### Atari Breakout

Breakout is a seminal arcade video game developed by Atari, Inc., and released in 1976. Designed by Nolan Bushnell and Steve Bristow, with engineering by Steve Wozniak, it evolved from Atari's Pong and became a cornerstone of early gaming.

**Gameplay**: Players control a horizontal paddle at the bottom of the screen to bounce a ball toward a wall of colored bricks at the top. The ball destroys bricks on contact, scoring points, while deflecting off walls and the paddle. The goal is to clear all bricks across levels without letting the ball fall past the paddle (losing a life). Power-ups and speed increases add challenge; it's single-player with simple controls (knob or joystick).

**Mechanics and Features**:
- **Core Loop**: Ball physics simulate realistic bounces; breaking bricks can accelerate the ball or introduce multiples.
- **Scoring**: Points vary by brick color/position; high scores encourage replay.
- **Lives and Progression**: Start with limited lives; levels advance with faster gameplay and varied brick layouts.
- **Hardware**: Originally an arcade cabinet with black-and-white display (color overlays added); ports to Atari 2600 (1978) popularized home versions.

**Legacy and Impact**: Breakout sold over 11,000 arcade units, grossing millions, and inspired the brick-breaker genre (e.g., Arkanoid, modern variants like Peggle). It influenced Apple's founding (Wozniak's involvement) and gaming design, emphasizing skill, precision, and endless replayability. Today, it's emulated widely and referenced in pop culture, symbolizing 1970s arcade innovation.
{{< /sidenote >}}

The paper introduces a novel agent, which they call a Deep Q-Network (DQN), that learns to play Atari games by looking only at the screen pixels, just like a human. The key innovations were:
{{< sidenote side="left" >}}![atari paper](/assets/images/atari-paper.png){{< /sidenote >}}

1.End-to-End Learning: Using a deep Convolutional Neural Network (CNN) to directly process raw pixels and output action values, eliminating the need for hand-crafted features.

2.Experience Replay: Storing the agent's experiences (state, action, reward, next state) in a memory buffer and then randomly sampling from this buffer to train the network. This breaks the correlation between consecutive samples and smooths the data distribution, stabilizing learning.


3.Target Network (A crucial detail from the follow-up 2015 paper, but implied here): Using a separate, fixed network to generate the target Q-values for the Bellman update. This prevents the target from rapidly changing, which further stabilizes training.

### lets build this components in python 
Prerequisites 

{{< sidenote >}}
torch:  The deep learning framework.
gymnasium:  The modern fork of OpenAI Gym, used for creating and interacting with the Atari environment.
ale-py:  The Atari Learning Environment (ALE) backend for Gymnasium.
numpy: For numerical operations.
opencv-python:  For efficient image preprocessing (grayscale, resizing)
{{< /sidenote >}}

```python 
pip install torch gymnasium[atari] ale-py numpy opencv-python
```


### Step 1: Setting up the Environment & Preprocessing
The paper specifies a series of preprocessing steps to make learning more manageable.
* Convert the image to grayscale.
* Downsample the image to 110x84.
* Crop the image to an 84x84 region that captures the play area.
* Stack the last 4 frames together to give the network a sense of motion (e.g., the ball's velocity).

⠀We can encapsulate this logic in a gymnasium wrapper.
{{< sidenote >}}
How to use it:
env = gym.make("ALE/Breakout-v5") 
Use the full name with "ALE/"
processed_env = AtariPreprocessing(env)
state, info = processed_env.reset()
print(f"Observation shape: {state.shape}") # Should be (4, 84, 84)
{{< /sidenote >}}

```python 
import gymnasium as gym
import cv2
import numpy as np
from collections import deque

class AtariPreprocessing(gym.Wrapper):
    """
    A wrapper for Atari environments that preprocesses observations as described
    in the DQN paper.
    - Grayscale and resize frames
    - Stack the last 4 frames
    """
    def __init__(self, env, frame_stack_size=4):
        super().__init__(env)
        self.frame_stack_size = frame_stack_size
        self.frames = deque([], maxlen=self.frame_stack_size)
        
        # Define the new observation space shape
        # The shape is (frame_stack_size, height, width)
        self.observation_space = gym.spaces.Box(
            low=0, high=255, 
            shape=(self.frame_stack_size, 84, 84), 
            dtype=np.uint8
        )

    def _preprocess_frame(self, frame):
        # Convert to grayscale
        frame = cv2.cvtColor(frame, cv2.COLOR_RGB2GRAY)
        # Resize to 110x84, then crop to 84x84
        frame = cv2.resize(frame, (84, 84), interpolation=cv2.INTER_AREA)
        return frame

    def reset(self, **kwargs):
        # Reset the underlying environment
        observation, info = self.env.reset(**kwargs)
        
        # Preprocess the first frame and fill the deque
        processed_frame = self._preprocess_frame(observation)
        for _ in range(self.frame_stack_size):
            self.frames.append(processed_frame)
            
        return self._get_observation(), info

    def step(self, action):
        observation, reward, terminated, truncated, info = self.env.step(action)
        
        # Preprocess the new frame and add it to the deque
        processed_frame = self._preprocess_frame(observation)
        self.frames.append(processed_frame)
        
        return self._get_observation(), reward, terminated, truncated, info

    def _get_observation(self):
        # Return the stacked frames as a single numpy array
        return np.array(self.frames)

```