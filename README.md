# workout-planner
Tech Stack:
Frontend: HTML, CSS, JavaScript (React.js)
Backend: Node.js (Express)
Database: MongoDB (or any NoSQL database)
# Database Schema:
{
  title: String,
  date: Date,
  exercises: [
    {
      name: String,
      sets: Number,
      reps: Number
    }
  ]
}
#Frontend Code
import React from 'react';

function HomePage() {
  return (
    <div className="container">
      <h1>Workout Planner</h1>
      <div>
        <button>Create New Workout</button>
        <button>View Workouts</button>
      </div>
    </div>
  );
}

export default HomePage;
import React, { useState } from 'react';

function CreateWorkout() {
  const [title, setTitle] = useState('');
  const [exercises, setExercises] = useState([{ name: '', sets: 0, reps: 0 }]);

  const handleExerciseChange = (index, e) => {
    const newExercises = [...exercises];
    newExercises[index][e.target.name] = e.target.value;
    setExercises(newExercises);
  };

  const addExercise = () => {
    setExercises([...exercises, { name: '', sets: 0, reps: 0 }]);
  };

  const handleSubmit = async () => {
    const workout = { title, exercises };
    await fetch('/api/workouts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(workout),
    });
    alert('Workout Created!');
  };

  return (
    <div className="container">
      <h2>Create Workout</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="Workout Title"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
        />
        {exercises.map((exercise, index) => (
          <div key={index}>
            <input
              name="name"
              type="text"
              placeholder="Exercise Name"
              value={exercise.name}
              onChange={(e) => handleExerciseChange(index, e)}
            />
            <input
              name="sets"
              type="number"
              placeholder="Sets"
              value={exercise.sets}
              onChange={(e) => handleExerciseChange(index, e)}
            />
            <input
              name="reps"
              type="number"
              placeholder="Reps"
              value={exercise.reps}
              onChange={(e) => handleExerciseChange(index, e)}
            />
          </div>
        ))}
        <button type="button" onClick={addExercise}>Add Exercise</button>
        <button type="submit">Create Workout</button>
      </form>
    </div>
  );
}

export default CreateWorkout;
#backend code
const express = require('express');
const mongoose = require('mongoose');
const app = express();

app.use(express.json());

mongoose.connect('mongodb://localhost:27017/workoutPlanner', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

const workoutSchema = new mongoose.Schema({
  title: String,
  date: Date,
  exercises: [
    {
      name: String,
      sets: Number,
      reps: Number,
    },
  ],
});

const Workout = mongoose.model('Workout', workoutSchema);

// Create a workout
app.post('/api/workouts', async (req, res) => {
  const workout = new Workout(req.body);
  await workout.save();
  res.status(201).send(workout);
});

// Get all workouts
app.get('/api/workouts', async (req, res) => {
  const workouts = await Workout.find();
  res.send(workouts);
});

// Get a single workout by ID
app.get('/api/workouts/:id', async (req, res) => {
  const workout = await Workout.findById(req.params.id);
  res.send(workout);
});

// Update a workout
app.put('/api/workouts/:id', async (req, res) => {
  const workout = await Workout.findByIdAndUpdate(req.params.id, req.body, { new: true });
  res.send(workout);
});

// Delete a workout
app.delete('/api/workouts/:id', async (req, res) => {
  await Workout.findByIdAndDelete(req.params.id);
  res.send({ message: 'Workout deleted' });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
