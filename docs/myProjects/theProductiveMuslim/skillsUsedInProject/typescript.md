# Typescript

I dont particularly enjoy frontend coding, but I wrote my frontend code in typescript, as it helps reduce bugs and errors to do with data types.

Heres an example of a function that I wrote that calculates what the current prayer is.

```javascript
interface PrayerData {
  Asr: string;
  Dhuhr: string;
  Fajr: string;
  Isha: string;
  Maghrib: string;
}

interface ClosestPrayer {
  name: string;
  time: string;
  difference: number;
}

// this function returns next prayer in closestPrayer interface
async function getCurrentPrayer(
  todaysPrayers: PrayerData
): Promise<ClosestPrayer> {
  const currTime = new Date();

  const filteredPrayerObj: Record<string, ClosestPrayer> = Object.entries(
    todaysPrayers
  ).reduce((acc, [key, value]) => {
    const prayerTime = new Date(value);

    if (prayerTime <= currTime) {
      const difference = currTime.getTime() - prayerTime.getTime();
      acc[key] = { name: key, time: value, difference };
    }

    return acc;
  }, {} as Record<string, ClosestPrayer>);
  console.log(filteredPrayerObj);

  // If there are no upcoming prayers, return null
  if (Object.keys(filteredPrayerObj).length === 0) {
    return {
      name: "null - no upcoming prayers",
      time: "null",
      difference: 0,
    };
  }

  const closestPrayer = Object.values(filteredPrayerObj).reduce(
    (closest, current) => {
      return current.difference < closest.difference ? current : closest;
    },
    { name: "", time: "", difference: Infinity }
  );

  return closestPrayer;
}

export default getCurrentPrayer;
```
