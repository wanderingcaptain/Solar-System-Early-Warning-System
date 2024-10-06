

# Solar System Simulation with Asteroid Proximity Alert

This project is an interactive solar system simulation built with Vue.js and X3DOM. It dynamically displays the orbits and positions of the eight major planets, near-Earth objects (NEOs), and comets. The app includes an asteroid proximity alert feature that detects when an asteroid is close to Earth and dynamically displays the asteroid's position and orbit.

## Features

- **3D Visualization of the Solar System**: Displays the orbits and positions of the eight major planets and other celestial objects.
- **Asteroid Proximity Alert**: Detects the proximity of near-Earth objects (NEOs) to Earth and dynamically shows or hides their orbits when they come within a specified distance.
- **Dynamic Updates**: Planet and asteroid positions are updated in real-time based on their orbital parameters.
- **Adjustable Simulation Speed**: Users can control the speed of the simulation using a slider (if enabled).
- **Toggle Orbit and Label Visibility**: Users can toggle the visibility of orbits and labels for a cleaner view of the simulation.

## Technologies Used

- **Vue.js**: Frontend framework used to build the interactive UI and manage the simulation state.
- **X3DOM**: JavaScript library for 3D graphics rendering, used to visualize the planets and orbits.
- **PapaParse**: CSV parsing library, used to load and process data for asteroids and other celestial objects.
- **NASA API**: Retrieves data for near-Earth objects and comets to include in the simulation.

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/solar-system-simulation.git
   cd solar-system-simulation
   ```

2. Install the project dependencies:
   ```bash
   npm install
   ```

3. Run the development server:
   ```bash
   npm run serve
   ```

4. Open your browser and go to `http://localhost:8080` to view the application.

## Usage

- **View Planets**: The eight major planets (Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, and Neptune) are always visible, along with their orbits.
- **Asteroid Proximity Alert**: If a near-Earth object (NEO) gets within a certain distance from Earth, it will appear with its orbit.
- **Control Simulation**: Users can modify the simulation speed using the speed slider (if implemented), and toggle orbit and label visibility using buttons (if enabled).

## Data Sources

- **NASA Small Body Database**: Provides orbital data for near-Earth objects and comets.
- **CSV Files**: Additional asteroid and celestial object data can be added via CSV files.

## Asteroid Proximity Alert Feature

The alert feature detects the proximity of asteroids to Earth and dynamically displays their position and orbit when they come within a specified distance. The proximity threshold can be adjusted via the `alertDistance` variable in the Vue component.

## Project Structure

```
/src
  ├── /components
  │   └── solarsystem.vue       # Main component for the solar system simulation
  ├── App.vue                   # Main application file
  └── main.js                   # Entry point for the Vue application
/public
  └── sbdb_query_results.csv     # Example CSV file with asteroid data
```

## Future Enhancements

- **Collision Detection**: Extend the proximity alert feature to include potential collision detection for asteroids that may intersect Earth's orbit.
- **Additional Celestial Objects**: Add more detailed visualizations for moons and dwarf planets.
- **Improved Performance**: Optimize the rendering of large numbers of objects, especially with dense asteroid fields.

## License

This project is licensed under the MIT License. See the `LICENSE` file for more information.
