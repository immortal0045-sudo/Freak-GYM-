# Freak-GYM-
   A simple fitness app for gym members who dont take personal training where they get daily workout routines with video demo can track sets /reps and mark workout as completed 
FitDaily/
├── app/
│   ├── (tabs)/
│   │   ├── _layout.tsx
│   │   ├── home.tsx
│   │   ├── workouts.tsx
│   │   ├── history.tsx
│   │   └── profile.tsx
│   ├── workout/[id].tsx
│   └── _layout.tsx
├── components/
│   ├── ExerciseCard.tsx
│   ├── WorkoutCard.tsx
│   └── ProgressCircle.tsx
├── store/
│   └── workoutStore.ts
├── data/
│   └── workouts.ts
├── constants/
│   └── colors.ts
├── tailwind.config.js
├── package.json
└── app.json
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./app/**/*.{js,jsx,ts,tsx}", "./components/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
}import { Slot } from 'expo-router';
import { SafeAreaProvider } from 'react-native-safe-area-context';

export default function RootLayout() {
  return (
    <SafeAreaProvider>
      <Slot />
    </SafeAreaProvider>
  );
}export const workouts = [
  {
    id: "1",
    day: "Day 1 - Push Day",
    focus: "Chest • Shoulders • Triceps",
    duration: "45-55 min",
    level: "Intermediate",
    exercises: [
      {
        id: "e1",
        name: "Bench Press",
        sets: 4,
        reps: "8-12",
        videoUrl: "https://www.youtube.com/embed/IOX2F8bQ8zM",
        demo: "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4" // Replace with real video
      },
      {
        id: "e2",
        name: "Overhead Shoulder Press",
        sets: 3,
        reps: "10-12",
        videoUrl: "https://www.youtube.com/embed/2yjwXTQ6S8M"
      },
      {
        id: "e3",
        name: "Tricep Dips",
        sets: 3,
        reps: "12-15",
        videoUrl: "https://www.youtube.com/embed/6kAL8v2f3cA"
      }
    ]
  }
];import { create } from 'zustand';

interface ExerciseLog {
  exerciseId: string;
  sets: number[];
  completed: boolean;
}

interface WorkoutStore {
  completedWorkouts: string[];
  workoutLogs: Record<string, ExerciseLog[]>;
  markWorkoutComplete: (workoutId: string) => void;
  logExercise: (workoutId: string, exerciseId: string, sets: number[]) => void;
}

export const useWorkoutStore = create<WorkoutStore>((set) => ({
  completedWorkouts: [],
  workoutLogs: {},

  markWorkoutComplete: (workoutId) =>
    set((state) => ({
      completedWorkouts: [...state.completedWorkouts, workoutId],
    })),

  logExercise: (workoutId, exerciseId, sets) =>
    set((state) => ({
      workoutLogs: {
        ...state.workoutLogs,
        [workoutId]: [
          ...(state.workoutLogs[workoutId] || []),
          { exerciseId, sets, completed: true },
        ],
      },
    })),
}));import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, Alert } from 'react-native';
import { Video, ResizeMode } from 'expo-av';
import { Check } from 'lucide-react-native';
import { useWorkoutStore } from '../store/workoutStore';

interface Props {
  exercise: any;
  workoutId: string;
}

export default function ExerciseCard({ exercise, workoutId }: Props) {
  const [setsData, setSetsData] = useState<number[]>(new Array(exercise.sets).fill(0));
  const [completed, setCompleted] = useState(false);
  const { logExercise } = useWorkoutStore();

  const handleComplete = () => {
    logExercise(workoutId, exercise.id, setsData);
    setCompleted(true);
    Alert.alert("Great Job!", `${exercise.name} completed!`);
  };

  return (
    <View className="bg-zinc-900 p-5 rounded-3xl mb-6">
      <Text className="text-white text-2xl font-bold">{exercise.name}</Text>
      <Text className="text-zinc-400 mt-1">
        {exercise.sets} sets × {exercise.reps}
      </Text>

      {/* Video Demo */}
      <View className="my-4 rounded-2xl overflow-hidden bg-black">
        <Video
          source={{ uri: exercise.demo }}
          className="w-full h-56"
          useNativeControls
          resizeMode={ResizeMode.CONTAIN}
          isLooping
        />
      </View>

      {/* Sets Input */}
      <View className="flex-row flex-wrap gap-3">
        {setsData.map((_, index) => (
          <View key={index} className="bg-zinc-800 px-4 py-3 rounded-xl flex-row items-center">
            <Text className="text-white font-medium">Set {index + 1}</Text>
            <TextInput
              className="bg-zinc-700 text-white w-14 text-center ml-3 rounded-lg py-1"
              keyboardType="numeric"
              placeholder="10"
              onChangeText={(text) => {
                const newSets = [...setsData];
                newSets[index] = parseInt(text) || 0;
                setSetsData(newSets);
              }}
            />
          </View>
        ))}
      </View>

      <TouchableOpacity
        onPress={handleComplete}
        disabled={completed}
        className={`mt-6 py-4 rounded-2xl flex-row justify-center items-center ${completed ? 'bg-green-600' : 'bg-green-500'}`}
      >
        <Check color="white" size={20} />
        <Text className="text-white font-bold text-lg ml-2">
          {completed ? "Completed ✓" : "Mark Exercise Complete"}
        </Text>
      </TouchableOpacity>
    </View>
  );
}import React from 'react';
import { ScrollView, Text, View, TouchableOpacity } from 'react-native';
import { useRouter } from 'expo-router';
import ExerciseCard from '../../components/ExerciseCard';
import { workouts } from '../../data/workouts';
import { useWorkoutStore } from '../../store/workoutStore';

export default function WorkoutsScreen() {
  const router = useRouter();
  const { completedWorkouts } = useWorkoutStore();
  const workout = workouts[0];

  const isCompleted = completedWorkouts.includes(workout.id);

  return (
    <ScrollView className="flex-1 bg-black px-4 pt-12">
      <Text className="text-4xl font-bold text-white">Today's Workout</Text>
      <Text className="text-zinc-400 text-lg mt-1">{workout.day}</Text>

      <View className="bg-zinc-900 p-4 rounded-2xl my-6">
        <Text className="text-white text-lg">{workout.focus}</Text>
        <Text className="text-emerald-400">{workout.duration}</Text>
      </View>

      {workout.exercises.map((ex) => (
        <ExerciseCard key={ex.id} exercise={ex} workoutId={workout.id} />
      ))}

      {isCompleted ? (
        <Text className="text-green-500 text-center text-xl font-bold py-8">
          🎉 Workout Completed Today!
        </Text>
      ) : (
        <TouchableOpacity
          onPress={() => alert("All exercises completed! Great work!")}
          className="bg-white py-5 rounded-3xl mt-4"
        >
          <Text className="text-black text-center font-bold text-xl">
            Finish Full Workout
          </Text>
        </TouchableOpacity>
      )}
    </ScrollView>
  );
}
