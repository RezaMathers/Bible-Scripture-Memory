I have created a React prototype that implements this organizational system and review methodology.



```tsx
import React, { useState } from 'react';
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Plus, ChevronRight } from 'lucide-react';

const ScriptureMemorySystem = () => {
  const [verses, setVerses] = useState({
    daily: [],
    odd: [],
    even: [],
    daysOfWeek: {
      sunday: [],
      monday: [],
      tuesday: [],
      wednesday: [],
      thursday: [],
      friday: [],
      saturday: []
    },
    numbered: Array(31).fill([]).map(() => [])
  });

  const [newVerse, setNewVerse] = useState({
    reference: '',
    text: '',
    translation: 'NKJV'
  });

  // Get current review verses based on date
  const getCurrentReviewVerses = () => {
    const today = new Date();
    const dayOfWeek = today.toLocaleDateString('en-US', { weekday: 'lowercase' });
    const date = today.getDate();
    const isOdd = date % 2 === 1;

    return {
      daily: verses.daily,
      alternating: isOdd ? verses.odd : verses.even,
      weekly: verses.daysOfWeek[dayOfWeek],
      monthly: verses.numbered[date - 1]
    };
  };

  // Advance a verse to the next stage
  const advanceVerse = (verse) => {
    setVerses(prev => {
      // Remove from daily
      const newDaily = prev.daily.filter(v => v !== verse);
      
      // Add to odd/even (alternating)
      const destinationSection = prev.odd.length <= prev.even.length ? 'odd' : 'even';
      const oldAlternatingVerse = prev[destinationSection][0];
      const newAlternating = [verse, ...prev[destinationSection].slice(1)];

      // Move old alternating verse to day of week
      const daysOfWeek = { ...prev.daysOfWeek };
      const weekdayKeys = Object.keys(daysOfWeek);
      const targetDay = weekdayKeys[Math.floor(Math.random() * 7)];
      const oldWeeklyVerse = daysOfWeek[targetDay][0];
      daysOfWeek[targetDay] = [oldAlternatingVerse, ...daysOfWeek[targetDay].slice(1)];

      // Move old weekly verse to numbered
      const numbered = [...prev.numbered];
      const targetNumber = Math.floor(Math.random() * 31);
      numbered[targetNumber] = [oldWeeklyVerse, ...numbered[targetNumber]];

      return {
        ...prev,
        daily: newDaily,
        [destinationSection]: newAlternating,
        daysOfWeek,
        numbered
      };
    });
  };

  // Add new verse to system
  const addNewVerse = () => {
    if (newVerse.reference && newVerse.text) {
      setVerses(prev => ({
        ...prev,
        daily: [...prev.daily, newVerse]
      }));
      setNewVerse({
        reference: '',
        text: '',
        translation: 'NKJV'
      });
    }
  };

  const reviewVerses = getCurrentReviewVerses();

  return (
    <div className="w-full max-w-4xl p-4 space-y-6">
      {/* Add New Verse Section */}
      <Card>
        <CardHeader>
          <CardTitle>Add New Verse</CardTitle>
        </CardHeader>
        <CardContent className="space-y-4">
          <Input
            placeholder="Reference (e.g., John 3:16)"
            value={newVerse.reference}
            onChange={e => setNewVerse(prev => ({ ...prev, reference: e.target.value }))}
          />
          <Input
            placeholder="Verse text"
            value={newVerse.text}
            onChange={e => setNewVerse(prev => ({ ...prev, text: e.target.value }))}
          />
          <Button onClick={addNewVerse}>
            <Plus className="w-4 h-4 mr-2" />
            Add Verse
          </Button>
        </CardContent>
      </Card>

      {/* Today's Review Section */}
      <Card>
        <CardHeader>
          <CardTitle>Today's Review</CardTitle>
        </CardHeader>
        <CardContent className="space-y-4">
          {/* Daily Verse */}
          <div className="space-y-2">
            <h3 className="font-semibold">Daily (New)</h3>
            {reviewVerses.daily.map((verse, idx) => (
              <div key={idx} className="p-4 border rounded-lg">
                <div className="flex justify-between items-center">
                  <div>
                    <p className="font-medium">{verse.reference}</p>
                    <p className="text-gray-600">{verse.text}</p>
                    <p className="text-sm text-gray-400">{verse.translation}</p>
                  </div>
                  <Button variant="outline" onClick={() => advanceVerse(verse)}>
                    <ChevronRight className="w-4 h-4" />
                  </Button>
                </div>
              </div>
            ))}
          </div>

          {/* Other Review Sections */}
          {['alternating', 'weekly', 'monthly'].map((section) => (
            <div key={section} className="space-y-2">
              <h3 className="font-semibold capitalize">{section}</h3>
              {reviewVerses[section]?.map((verse, idx) => (
                <div key={idx} className="p-4 border rounded-lg">
                  <p className="font-medium">{verse.reference}</p>
                  <p className="text-gray-600">{verse.text}</p>
                  <p className="text-sm text-gray-400">{verse.translation}</p>
                </div>
              ))}
            </div>
          ))}
        </CardContent>
      </Card>
    </div>
  );
};

export default ScriptureMemorySystem;

```

I've created a React component that implements the Scripture Memory System you described. Here are the key features:

1. Organization System:
- Daily verses (currently being memorized)
- Odd/Even day verses
- Days of the week verses
- Monthly verses (1-31)

2. Core Functionality:
- Add new verses with reference, text, and translation
- Automatically shows today's verses based on:
  - Daily verse(s)
  - Odd/Even day verses (based on current date)
  - Day of the week verses
  - Date of the month verses
- Advance verses through the system with a single click
- Automatic rotation of verses through different categories

3. User Interface:
- Clean, card-based layout using shadcn/ui components
- Easy input form for new verses
- Clear organization of daily review verses
- One-click advancement of memorized verses
