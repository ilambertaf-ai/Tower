import React, { useState, useEffect } from 'react';
import { Calendar, Menu, X, ChevronLeft, ChevronRight, Moon, Sun, ArrowLeft, Plus, Trash2 } from 'lucide-react';

const FitnessApp = () => {
  const [darkMode, setDarkMode] = useState(false);
  const [currentView, setCurrentView] = useState('calendar');
  const [menuOpen, setMenuOpen] = useState(false);
  const [currentDate, setCurrentDate] = useState(new Date());
  const [selectedDate, setSelectedDate] = useState(null);
  const [calendarData, setCalendarData] = useState({});
  const [showDayModal, setShowDayModal] = useState(false);
  
  // Modal state for day details
  const [workoutPlan, setWorkoutPlan] = useState([]);
  const [foodEntries, setFoodEntries] = useState([]);
  const [dailyMetrics, setDailyMetrics] = useState({
    weight: '',
    sleep: '',
    water: '',
    calories: ''
  });

  const getDaysInMonth = (date) => {
    const year = date.getFullYear();
    const month = date.getMonth();
    const firstDay = new Date(year, month, 1);
    const lastDay = new Date(year, month + 1, 0);
    const daysInMonth = lastDay.getDate();
    const startingDayOfWeek = firstDay.getDay();
    
    return { daysInMonth, startingDayOfWeek };
  };

  const formatDateKey = (date) => {
    return `${date.getFullYear()}-${date.getMonth()}-${date.getDate()}`;
  };

  const getCalendarDataForDate = (date) => {
    const key = formatDateKey(date);
    return calendarData[key] || null;
  };

  const handleDateClick = (day) => {
    const clickedDate = new Date(currentDate.getFullYear(), currentDate.getMonth(), day);
    setSelectedDate(clickedDate);
    
    const existingData = getCalendarDataForDate(clickedDate);
    if (existingData) {
      setWorkoutPlan(existingData.workouts || []);
      setFoodEntries(existingData.food || []);
      setDailyMetrics(existingData.metrics || { weight: '', sleep: '', water: '', calories: '' });
    } else {
      setWorkoutPlan([]);
      setFoodEntries([]);
      setDailyMetrics({ weight: '', sleep: '', water: '', calories: '' });
    }
    
    setShowDayModal(true);
  };

  const saveDayData = () => {
    const key = formatDateKey(selectedDate);
    const newData = {
      ...calendarData,
      [key]: {
        workouts: workoutPlan,
        food: foodEntries,
        metrics: dailyMetrics,
        saved: true,
        date: selectedDate.toISOString()
      }
    };
    setCalendarData(newData);
    setShowDayModal(false);
    alert('Day saved successfully!');
  };

  const addWorkout = () => {
    setWorkoutPlan([...workoutPlan, { exercise: '', sets: '', reps: '' }]);
  };

  const updateWorkout = (index, field, value) => {
    const updated = [...workoutPlan];
    updated[index][field] = value;
    setWorkoutPlan(updated);
  };

  const removeWorkout = (index) => {
    setWorkoutPlan(workoutPlan.filter((_, i) => i !== index));
  };

  const addFood = () => {
    setFoodEntries([...foodEntries, { name: '', calories: '' }]);
  };

  const updateFood = (index, field, value) => {
    const updated = [...foodEntries];
    updated[index][field] = value;
    setFoodEntries(updated);
  };

  const removeFood = (index) => {
    setFoodEntries(foodEntries.filter((_, i) => i !== index));
  };

  const calculateHealthAverages = () => {
    const savedDays = Object.values(calendarData).filter(d => d.saved && d.metrics);
    const count = savedDays.length;
    
    if (count === 0) return { weight: 0, sleep: 0, water: 0, calories: 0 };
    
    const totals = savedDays.reduce((acc, day) => ({
      weight: acc.weight + (parseFloat(day.metrics.weight) || 0),
      sleep: acc.sleep + (parseFloat(day.metrics.sleep) || 0),
      water: acc.water + (parseFloat(day.metrics.water) || 0),
      calories: acc.calories + (parseFloat(day.metrics.calories) || 0)
    }), { weight: 0, sleep: 0, water: 0, calories: 0 });

    return {
      weight: (totals.weight / count).toFixed(1),
      sleep: (totals.sleep / count).toFixed(1),
      water: (totals.water / count).toFixed(0),
      calories: (totals.calories / count).toFixed(0)
    };
  };

  const getMostRecentMetrics = () => {
    const savedDays = Object.entries(calendarData)
      .filter(([_, d]) => d.saved && d.metrics)
      .sort(([_, a], [__, b]) => new Date(b.date) - new Date(a.date));
    
    return savedDays.length > 0 ? savedDays[0][1] : null;
  };

  const getTopExerciseMetrics = () => {
    const exercises = {};
    Object.values(calendarData).forEach(day => {
      if (day.workouts) {
        day.workouts.forEach(w => {
          const key = w.exercise.toLowerCase();
          if (!exercises[key] || parseInt(w.sets) * parseInt(w.reps) > exercises[key].total) {
            exercises[key] = {
              name: w.exercise,
              sets: w.sets,
              reps: w.reps,
              total: parseInt(w.sets) * parseInt(w.reps)
            };
          }
        });
      }
    });
    return Object.values(exercises).slice(0, 5);
  };

  const getNextPlannedWorkout = () => {
    const today = new Date();
    const futureDays = Object.entries(calendarData)
      .filter(([_, d]) => new Date(d.date) >= today && d.workouts && d.workouts.length > 0)
      .sort(([_, a], [__, b]) => new Date(a.date) - new Date(b.date));
    
    return futureDays.length > 0 ? futureDays[0][1] : null;
  };

  const renderCalendar = () => {
    const { daysInMonth, startingDayOfWeek } = getDaysInMonth(currentDate);
    const days = [];
    
    for (let i = 0; i < startingDayOfWeek; i++) {
      days.push(<div key={`empty-${i}`} className="aspect-square" />);
    }
    
    for (let day = 1; day <= daysInMonth; day++) {
      const date = new Date(currentDate.getFullYear(), currentDate.getMonth(), day);
      const hasData = getCalendarDataForDate(date);
      
      days.push(
        <div
          key={day}
          onClick={() => handleDateClick(day)}
          className={`aspect-square flex flex-col items-center justify-center border cursor-pointer transition-colors
            ${darkMode ? 'border-gray-700 hover:bg-gray-700' : 'border-gray-200 hover:bg-gray-100'}
            ${hasData ? (darkMode ? 'bg-blue-900' : 'bg-blue-50') : ''}`}
        >
          <span className="text-sm">{day}</span>
          {hasData && <div className="w-1 h-1 rounded-full bg-blue-500 mt-1" />}
        </div>
      );
    }
    
    return days;
  };

  const bgColor = darkMode ? 'bg-gray-900' : 'bg-white';
  const textColor = darkMode ? 'text-white' : 'text-gray-900';
  const borderColor = darkMode ? 'border-gray-700' : 'border-gray-200';

  return (
    <div className={`min-h-screen ${bgColor} ${textColor} transition-colors`}>
      {/* Header */}
      <div className={`flex items-center justify-between p-4 border-b ${borderColor}`}>
        {currentView !== 'calendar' && (
          <button onClick={() => setCurrentView('calendar')} className="p-2">
            <ArrowLeft size={24} />
          </button>
        )}
        <h1 className="text-xl font-bold flex-1">
          {currentView === 'calendar' && 'Fitness Tracker'}
          {currentView === 'health' && 'Health Metrics'}
          {currentView === 'exercise' && 'Exercise Stats'}
          {currentView === 'data' && 'All Data'}
        </h1>
        <button onClick={() => setMenuOpen(!menuOpen)} className="p-2">
          {menuOpen ? <X size={24} /> : <Menu size={24} />}
        </button>
      </div>

      {/* Menu Dropdown */}
      {menuOpen && (
        <div className={`absolute right-0 top-16 ${darkMode ? 'bg-gray-800' : 'bg-white'} border ${borderColor} rounded-lg shadow-lg z-50 w-48`}>
          <button
            onClick={() => { setCurrentView('health'); setMenuOpen(false); }}
            className="w-full text-left px-4 py-3 hover:bg-opacity-80"
          >
            Health
          </button>
          <button
            onClick={() => { setCurrentView('exercise'); setMenuOpen(false); }}
            className="w-full text-left px-4 py-3 hover:bg-opacity-80"
          >
            Exercise
          </button>
          <button
            onClick={() => { setCurrentView('data'); setMenuOpen(false); }}
            className="w-full text-left px-4 py-3 hover:bg-opacity-80"
          >
            Data
          </button>
          <div className="border-t border-gray-300 my-1" />
          <button
            onClick={() => { setDarkMode(!darkMode); setMenuOpen(false); }}
            className="w-full text-left px-4 py-3 hover:bg-opacity-80 flex items-center gap-2"
          >
            {darkMode ? <Sun size={18} /> : <Moon size={18} />}
            {darkMode ? 'Light Mode' : 'Dark Mode'}
          </button>
        </div>
      )}

      {/* Calendar View */}
      {currentView === 'calendar' && (
        <div className="p-4">
          <div className="flex items-center justify-between mb-4">
            <button onClick={() => setCurrentDate(new Date(currentDate.getFullYear(), currentDate.getMonth() - 1))}>
              <ChevronLeft size={24} />
            </button>
            <h2 className="text-lg font-semibold">
              {currentDate.toLocaleDateString('en-US', { month: 'long', year: 'numeric' })}
            </h2>
            <button onClick={() => setCurrentDate(new Date(currentDate.getFullYear(), currentDate.getMonth() + 1))}>
              <ChevronRight size={24} />
            </button>
          </div>
          
          <div className="grid grid-cols-7 gap-1 mb-2">
            {['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'].map(day => (
              <div key={day} className="text-center text-sm font-semibold py-2">{day}</div>
            ))}
          </div>
          
          <div className="grid grid-cols-7 gap-1">
            {renderCalendar()}
          </div>
        </div>
      )}

      {/* Health View */}
      {currentView === 'health' && (
        <div className="p-4 space-y-4">
          <div className={`p-4 border ${borderColor} rounded-lg`}>
            <h3 className="font-semibold mb-3">Current Averages</h3>
            {(() => {
              const avg = calculateHealthAverages();
              return (
                <div className="grid grid-cols-2 gap-3">
                  <div><span className="font-medium">Weight:</span> {avg.weight} lbs</div>
                  <div><span className="font-medium">Sleep:</span> {avg.sleep} hrs</div>
                  <div><span className="font-medium">Water:</span> {avg.water} oz</div>
                  <div><span className="font-medium">Calories:</span> {avg.calories}</div>
                </div>
              );
            })()}
          </div>
          
          <div className={`p-4 border ${borderColor} rounded-lg`}>
            <h3 className="font-semibold mb-3">Most Recent Entry</h3>
            {(() => {
              const recent = getMostRecentMetrics();
              if (!recent) return <p className="text-gray-500">No data yet</p>;
              return (
                <>
                  <p className="text-sm mb-2">{new Date(recent.date).toLocaleDateString()}</p>
                  <div className="grid grid-cols-2 gap-3">
                    <div><span className="font-medium">Weight:</span> {recent.metrics.weight} lbs</div>
                    <div><span className="font-medium">Sleep:</span> {recent.metrics.sleep} hrs</div>
                    <div><span className="font-medium">Water:</span> {recent.metrics.water} oz</div>
                    <div><span className="font-medium">Calories:</span> {recent.metrics.calories}</div>
                  </div>
                </>
              );
            })()}
          </div>
        </div>
      )}

      {/* Exercise View */}
      {currentView === 'exercise' && (
        <div className="p-4 space-y-4">
          <div className={`p-4 border ${borderColor} rounded-lg`}>
            <h3 className="font-semibold mb-3">Next Planned Workout</h3>
            {(() => {
              const next = getNextPlannedWorkout();
              if (!next) return <p className="text-gray-500">No upcoming workouts</p>;
              return (
                <>
                  <p className="text-sm mb-2">{new Date(next.date).toLocaleDateString()}</p>
                  {next.workouts.map((w, i) => (
                    <div key={i} className="mb-2">
                      {w.exercise}: {w.sets} sets × {w.reps} reps
                    </div>
                  ))}
                </>
              );
            })()}
          </div>
          
          <div className={`p-4 border ${borderColor} rounded-lg`}>
            <h3 className="font-semibold mb-3">Top Performance</h3>
            {getTopExerciseMetrics().map((ex, i) => (
              <div key={i} className="mb-2">
                <span className="font-medium">{ex.name}:</span> {ex.sets} sets × {ex.reps} reps
              </div>
            ))}
          </div>
        </div>
      )}

      {/* Data View */}
      {currentView === 'data' && (
        <div className="p-4 space-y-4">
          {Object.entries(calendarData)
            .filter(([_, d]) => d.saved)
            .sort(([_, a], [__, b]) => new Date(b.date) - new Date(a.date))
            .map(([key, day]) => (
              <div key={key} className={`p-4 border ${borderColor} rounded-lg`}>
                <h3 className="font-semibold mb-2">{new Date(day.date).toLocaleDateString()}</h3>
                
                {day.workouts && day.workouts.length > 0 && (
                  <div className="mb-3">
                    <p className="text-sm font-medium mb-1">Workouts:</p>
                    {day.workouts.map((w, i) => (
                      <div key={i} className="text-sm ml-2">• {w.exercise}: {w.sets}×{w.reps}</div>
                    ))}
                  </div>
                )}
                
                {day.metrics && (
                  <div className="text-sm grid grid-cols-2 gap-2">
                    {day.metrics.weight && <div>Weight: {day.metrics.weight} lbs</div>}
                    {day.metrics.sleep && <div>Sleep: {day.metrics.sleep} hrs</div>}
                    {day.metrics.water && <div>Water: {day.metrics.water} oz</div>}
                    {day.metrics.calories && <div>Calories: {day.metrics.calories}</div>}
                  </div>
                )}
              </div>
            ))}
        </div>
      )}

      {/* Day Modal */}
      {showDayModal && (
        <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
          <div className={`${bgColor} rounded-lg max-w-md w-full max-h-[90vh] overflow-y-auto p-6`}>
            <h2 className="text-xl font-bold mb-4">
              {selectedDate?.toLocaleDateString()}
            </h2>
            
            {/* Workouts */}
            <div className="mb-6">
              <div className="flex justify-between items-center mb-2">
                <h3 className="font-semibold">Workouts</h3>
                <button onClick={addWorkout} className="text-blue-500">
                  <Plus size={20} />
                </button>
              </div>
              {workoutPlan.map((workout, i) => (
                <div key={i} className={`flex gap-2 mb-2 p-2 border ${borderColor} rounded`}>
                  <input
                    placeholder="Exercise"
                    value={workout.exercise}
                    onChange={(e) => updateWorkout(i, 'exercise', e.target.value)}
                    className={`flex-1 px-2 py-1 rounded ${darkMode ? 'bg-gray-800' : 'bg-gray-100'}`}
                  />
                  <input
                    placeholder="Sets"
                    type="number"
                    value={workout.sets}
                    onChange={(e) => updateWorkout(i, 'sets', e.target.value)}
                    className={`w-16 px-2 py-1 rounded ${darkMode ? 'bg-gray-800' : 'bg-gray-100'}`}
                  />
                  <input
                    placeholder="Reps"
                    type="number"
                    value={workout.reps}
                    onChange={(e) => updateWorkout(i, 'reps', e.target.value)}
                    className={`w-16 px-2 py-1 rounded ${darkMode ? 'bg-gray-800' : 'bg-gray-100'}`}
                  />
                  <button onClick={() => removeWorkout(i)} className="text-red-500">
                    <Trash2 size={18} />
                  </button>
                </div>
              ))}
            </div>

            {/* Food */}
            <div className="mb-6">
              <div className="flex justify-between items-center mb-2">
                <h3 className="font-semibold">Food & Calories</h3>
                <button onClick={addFood} className="text-blue-500">
                  <Plus size={20} />
                </button>
              </div>
              {foodEntries.map((food, i) => (
                <div key={i} className={`flex gap-2 mb-2 p-2 border ${borderColor} rounded`}>
                  <input
                    placeholder="Food item"
                    value={food.name}
                    onChange={(e) => updateFood(i, 'name', e.target.value)}
                    className={`flex-1 px-2 py-1 rounded ${darkMode ? 'bg-gray-800' : 'bg-gray-100'}`}
                  />
                  <input
                    placeholder="Cal"
                    type="number"
                    value={food.calories}
                    onChange={(e) => updateFood(i, 'calories', e.target.value)}
                    className={`w-20 px-2 py-1 rounded ${darkMode ? 'bg-gray-800' : 'bg-gray-100'}`}
                  />
                  <button onClick={() => removeFood(i)} className="text-red-500">
                    <Trash2 size={18} />
                  </button>
                </div>
              ))}
            </div>

            {/* Daily Metrics */}
            <div className="mb-6">
              <h3 className="font-semibold mb-2">Daily Metrics</h3>
              <div className="space-y-2">
                <input
                  placeholder="Weight (lbs)"
                  type="number"
                  value={dailyMetrics.weight}
                  onChange={(e) => setDailyMetrics({...dailyMetrics, weight: e.target.value})}
                  className={`w-full px-3 py-2 rounded ${darkMode ? 'bg-gray-800' : 'bg-gray-100'}`}
                />
                <input
                  placeholder="Sleep (hours)"
                  type="number"
                  value={dailyMetrics.sleep}
                  onChange={(e) => setDailyMetrics({...dailyMetrics, sleep: e.target.value})}
                  className={`w-full px-3 py-2 rounded ${darkMode ? 'bg-gray-800' : 'bg-gray-100'}`}
                />
                <input
                  placeholder="Water (oz)"
                  type="number"
                  value={dailyMetrics.water}
                  onChange={(e) => setDailyMetrics({...dailyMetrics, water: e.target.value})}
                  className={`w-full px-3 py-2 rounded ${darkMode ? 'bg-gray-800' : 'bg-gray-100'}`}
                />
                <input
                  placeholder="Total Calories"
                  type="number"
                  value={dailyMetrics.calories}
                  onChange={(e) => setDailyMetrics({...dailyMetrics, calories: e.target.value})}
                  className={`w-full px-3 py-2 rounded ${darkMode ? 'bg-gray-800' : 'bg-gray-100'}`}
                />
              </div>
            </div>

            {/* Buttons */}
            <div className="flex gap-3">
              <button
                onClick={() => setShowDayModal(false)}
                className={`flex-1 py-2 rounded border ${borderColor}`}
              >
                Cancel
              </button>
              <button
                onClick={saveDayData}
                className="flex-1 py-2 rounded bg-blue-500 text-white"
              >
                Save Day
              </button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
};

export default FitnessApp;
