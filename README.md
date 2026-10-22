#include <iostream>
#include <fstream>
#include <iomanip>
#include <cmath>
#include <string>

// ============================================================================

constexpr char LAB_METHOD = 'B';

// --- Track Environment ---
constexpr double GRAVITY     = 9.80665; // Acceleration due to gravity (m/s^2)
constexpr double TRACK_ANGLE = 16.65;     // Incline angle of the track (in degrees)
constexpr double DISTANCE    = 0.746;     // Distance d between photogates or total roll length (m)

// --- Method B Specific Parameter ---
// Effective width w (m) as seen by the photogate (FLAG SIZE)
constexpr double EFFECTIVE_WIDTH = 0.02541;

// --- Object-Specific Parameters ---
// solid sphere
constexpr double SPHERE_MASS   = 0.066715;   // Mass in kg
constexpr double SPHERE_RADIUS = (0.02534 / 2);  // Outer radius in m

// solid cylinder
constexpr double CYLINDER_MASS   = 0.03244;   // Mass in kg
constexpr double CYLINDER_RADIUS = (0.02541 / 2);  // Outer radius in m

// hollow cylinder (thick-walled)
constexpr double HOLLOW_MASS       = 0.03185;  // Mass in kg
constexpr double HOLLOW_RADIUS_OUT = (0.02533 / 2);  // Outer radius in m
constexpr double HOLLOW_RADIUS_IN  = (0.02045 / 2);  // Inner radius in m


// Output destination file name
const std::string OUTPUT_FILE = "lab_predictions_1.txt";

// ============================================================================
// CORE PROCESSING AND MATH FUNCTIONS
// ============================================================================

// Convert degrees to radians
inline double deg_to_rad(double degrees) {
    return degrees * M_PI / 180.0;
}

struct RollingObject {
    std::string name;
    double mass;
    double r_out;
    double beta;
    double i_cm;
};

void process_and_log(std::ostream& os, const RollingObject& obj, double angle_rad) {

    // calculate linear acceleration down the track: a = (g * sin(theta)) / (1 + beta)
    double accel = (GRAVITY * std::sin(angle_rad)) / (1.0 + obj.beta);

    // calculate final translational velocity at the bottom: v = sqrt(2 * a * d)
    double v_final = std::sqrt(2.0 * accel * DISTANCE);

    // Method B: t = w / v_final
    double predicted_time = EFFECTIVE_WIDTH / v_final;;

    os << "--------------------------------------------------------------------\n";
    os << "Object: " << obj.name << "\n";
    os << "  - Mass (M)          : " << std::fixed << std::setprecision(4) << obj.mass << " kg\n";
    os << "  - Outer Radius (R)  : " << obj.r_out << " m\n";
    os << "  - Inertia Factor (b): " << obj.beta << "\n";
    os << "  - I_cm              : " << std::scientific << std::setprecision(4) << obj.i_cm << " kg*m^2\n";
    os << "  - Linear Accel (a)  : " << std::fixed << std::setprecision(4) << accel << " m/s^2\n";
    os << "  - Final Velocity (v): " << v_final << " m/s\n";
    os << "  * PREDICTED TIME    : " << std::setprecision(6) << predicted_time << " s\n";
}

int main() {
    double angle_rad = deg_to_rad(TRACK_ANGLE);

    // solid Sphere Beta = 2/5 = 0.4
    double sphere_beta = 0.4;
    double sphere_icm = sphere_beta * SPHERE_MASS * SPHERE_RADIUS * SPHERE_RADIUS;
    RollingObject sphere = {"Solid Sphere", SPHERE_MASS, SPHERE_RADIUS, sphere_beta, sphere_icm};

    // solid Cylinder Beta = 1/2 = 0.5
    double cylinder_beta = 0.5;
    double cylinder_icm = cylinder_beta * CYLINDER_MASS * CYLINDER_RADIUS * CYLINDER_RADIUS;
    RollingObject cylinder = {"Solid Cylinder", CYLINDER_MASS, CYLINDER_RADIUS, cylinder_beta, cylinder_icm};

    // thick-walled Hollow Cylinder Beta = 0.5 * (1 + R_in^2 / R_out^2)
    double hollow_beta = 0.5 * (1.0 + (HOLLOW_RADIUS_IN * HOLLOW_RADIUS_IN) / (HOLLOW_RADIUS_OUT * HOLLOW_RADIUS_OUT));
    double hollow_icm = 0.5 * HOLLOW_MASS * (HOLLOW_RADIUS_IN * HOLLOW_RADIUS_IN + HOLLOW_RADIUS_OUT * HOLLOW_RADIUS_OUT);
    RollingObject hollow = {"Hollow Cylinder (Thick-Walled)", HOLLOW_MASS, HOLLOW_RADIUS_OUT, hollow_beta, hollow_icm};

    // setup output file stream
    std::ofstream file_out(OUTPUT_FILE);
    if (!file_out.is_open()) {
        std::cerr << "Error: Could not open file " << OUTPUT_FILE << " for writing.\n";
        return 1;
    }

    // header generator function for both outputs
    auto print_header = [](std::ostream& os) {
        os << "====================================================================\n";
        os << "                 ROLLING MOTION PREDICTION RESULTS                  \n";
        os << "====================================================================\n";
        os << "Track Config: Angle = " << TRACK_ANGLE << " deg | Distance = " << DISTANCE << " m\n";
        os << "Method Selected: Method " << LAB_METHOD << " ("
           << ((LAB_METHOD == 'A' || LAB_METHOD == 'a') ? "2 Photogates - Travel Time" : "1 Photogate - Gate Blocking Time")
           << ")\n";
        if (LAB_METHOD != 'A' && LAB_METHOD != 'a') {
            os << "Effective Optical Width (w): " << EFFECTIVE_WIDTH << " m\n";
        }
    };

    // write to console terminal
    print_header(std::cout);
    process_and_log(std::cout, sphere, angle_rad);
    process_and_log(std::cout, cylinder, angle_rad);
    process_and_log(std::cout, hollow, angle_rad);
    std::cout << "--------------------------------------------------------------------\n";
    std::cout << "\n>>> Results successfully exported to " << OUTPUT_FILE << "\n\n";

    // write exact copies directly to the clean text file
    print_header(file_out);
    process_and_log(file_out, sphere, angle_rad);
    process_and_log(file_out, cylinder, angle_rad);
    process_and_log(file_out, hollow, angle_rad);
    file_out << "--------------------------------------------------------------------\n";

    file_out.close();

    return 0;
}

