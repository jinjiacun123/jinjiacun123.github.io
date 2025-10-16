```
//g++ main.cpp -o sfml-app -lsfml-graphics -lsfml-window -lsfml-system
#include <SFML/Graphics.hpp>

int main() {
// Create a window with dimensions 640x480 and title "SFML Window"
sf::RenderWindow window(sf::VideoMode(640, 480), "SFML Window");

// Create a circle shape with radius 50
sf::CircleShape circle(50);
circle.setFillColor(sf::Color::Green); // Set the color of the circle
circle.setPosition(295, 215); // Position the circle in the center

// Main loop to keep the window open
while (window.isOpen()) {
sf::Event event;
while (window.pollEvent(event)) {
if (event.type == sf::Event::Closed) // Close window event
window.close();
}

window.clear(); // Clear the screen
window.draw(circle); // Draw the circle
window.display(); // Display the updated frame
}

return 0;
}
```
