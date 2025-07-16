<!DOCTYPE html>
<html lang="en" dir="ltr">
<head>
    <script>
      // These global variables are provided by the Canvas environment.
      // Do not remove them or modify their assignment.
      var __app_id = "electricity-meters-system"; // Application ID
      var __firebase_config = JSON.stringify({
  apiKey: "AIzaSyDDl5vhcSpxqG7bKJRl_5WMk7f0GiOVqsk",
  authDomain: "meters-readings.firebaseapp.com",
  projectId: "meters-readings",
  storageBucket: "meters-readings.firebaseastorage.app",
  messagingSenderId: "487779544659",
  appId: "1:487779544659:web:b15d9a7585358b2c3360a6"
      });
    </script>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meters Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Import PDF libraries -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://unpkg.com/jspdf-autotable@3.8.2/dist/jspdf.plugin.autotable.js"></script>

    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">
    <!-- Firebase SDK imports updated to version 11.6.1 -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        // collection and onSnapshot exposed for global access (window)
        import { getFirestore, collection, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Make Firebase instances and core Firestore functions globally available
        window.firebaseApp = initializeApp(JSON.parse(__firebase_config), "metersApp");
        window.db = getFirestore(window.firebaseApp);
        window.auth = getAuth(window.firebaseApp);
        window.firestoreCollection = collection;
        window.firestoreOnSnapshot = onSnapshot;


        // Ensure initial authentication state is handled
        window.authReadyPromise = new Promise(resolve => {
            const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
            if (initialAuthToken) {
                signInWithCustomToken(window.auth, initialAuthToken)
                    .then(() => { console.log("DEBUG: Firebase Init Status: User is logged in with custom token."); resolve(); })
                    .catch(error => { console.warn("DEBUG: Firebase Init Error: Error signing in with custom token:", error); signInAnonymously(window.auth).then(() => { console.log("DEBUG: Firebase Init Status: Signed in anonymously after custom token failed."); resolve(); }); });
            } else {
                signInAnonymously(window.auth)
                    .then(() => { console.log("DEBUG: Firebase Init Status: Signed in anonymously."); resolve(); })
                    .catch(error => { console.error("DEBUG: Firebase Init Error: Anonymous sign-in failed:", error); resolve(); });
            }
        });
    </script>
    <style>
        body {
            background-color: #f0f4f8; /* Light and calm background color (light blue-gray) */
            color: #4a5568; /* Dark text color for better readability on light background */
            /* New geometric pattern with very subtle light gray color, increased size and clarity */
            background-image: repeating-conic-gradient(from 0deg at 50% 50%, rgba(200, 200, 200, 0.1) 0deg 45deg, transparent 45deg 90deg);
            background-size: 400px 400px; /* Increased to 5 times the previous size */
        }
        body.lang-en { font-family: 'Inter', sans-serif; }
        /* Dark mode styles */
        body.dark-mode {
            background-color: #1a202c; /* Dark background */
            color: #e2e8f0; /* Light text */
            /* Adjusted geometric pattern for dark mode: lighter gray for visibility */
            background-image: repeating-conic-gradient(from 0deg at 50% 50%, rgba(100, 100, 100, 0.1) 0deg 45deg, transparent 45deg 90deg); /* Lighter gray for dark mode pattern */
        }
        body.dark-mode .card { background-color: #2d3748; border-color: #4a5568; box-shadow: 0 4px 6px rgba(0,0,0,0.3); }
        body.dark-mode .nav-btn { background-color: #4a5568; border-color: #64748b; color: #e2e8f0; }
        body.dark-mode .nav-btn:hover:not(:disabled) { background-color: #5a67d8; }
        body.dark-mode .loader { border-color: #4a5568; border-top-color: #58a6ff; }
        body.dark-mode .view-image-btn:disabled { color: #64748b; }
        body.dark-mode .view-image-btn:not(:disabled) { color: #79c0ff; }
        body.dark-mode .view-image-btn:not(:disabled):hover { color: #90cdf4; }
        body.dark-mode .select-filter { background-color: #2d3748; border-color: #4a5568; color: #e2e8f0; }
        body.dark-mode .toggle-btn { box-shadow: 0 2px 4px rgba(0, 0, 0, 0.4); }
        body.dark-mode .summary-total-value { color: #e2e8f0; }
        body.dark-mode .summary-label { color: #a0aec0; }
        body.dark-mode .meter-reading-display { color: #e2e8f0; }
        body.dark-mode .meter-consumption-display { color: #e2e8f0; }
        body.dark-mode .view-meter-image-btn { color: #79c0ff; }
        body.dark-mode .view-meter-image-btn:hover { color: #90cdf4; }
        body.dark-mode .text-beige-icon { color: #d4a762; } /* Lighter beige for dark mode */
        body.dark-mode .border-beige-meter-card { border-color: #d4a762; }
        body.dark-mode .bg-beige-consumption-ashghal { background-color: rgba(212, 167, 98, 0.3); } /* Slightly more opaque for dark mode */
        body.dark-mode h1, body.dark-mode h2, body.dark-mode h3, body.dark-mode p, body.dark-mode span, body.dark-mode div { color: inherit; } /* Ensure text color inherits from body */
        body.dark-mode .border-yellow-500 { border-color: #f6e05e; } /* Lighter yellow for dark mode */
        body.dark-mode .border-blue-500 { border-color: #63b3ed; } /* Lighter blue for dark mode */
        body.dark-mode .text-yellow-700 { color: #f6e05e; } /* Lighter yellow for dark mode */
        body.dark-mode .text-blue-700 { color: #63b3ed; } /* Lighter blue for dark mode */
        body.dark-mode .bg-yellow-100 { background-color: rgba(246, 224, 94, 0.2); } /* Lighter yellow background for dark mode */
        body.dark-mode .bg-blue-100 { background-color: rgba(99, 179, 237, 0.2); } /* Lighter blue background for dark mode */
        body.dark-mode .text-gray-800 { color: #e2e8f0; }
        body.dark-mode .text-gray-600 { color: #a0aec0; }
        body.dark-mode .text-gray-500 { color: #a0aec0; }
        body.dark-mode .text-gray-900 { color: #e2e8f0; }


        .card { background-color: #ffffff; border: 1px solid #e2e8f0; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        .nav-btn { background-color: #e2e8f0; border: 1px solid #cbd5e1; color: #4a5568; }
        .nav-btn:hover:not(:disabled) { background-color: #cfd8e3; }
        .nav-btn:disabled { opacity: 0.5; cursor: not-allowed; }
        .loader { border: 4px solid #cbd5e1; border-top: 4px solid #58a6ff; border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; margin: 20px auto; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        .view-image-btn:disabled { color: #94a3b8; cursor: not-allowed; }
        .view-image-btn:not(:disabled) { color: #58a6ff; cursor: pointer; transition: color 0.2s;}
        .view-image-btn:not(:disabled):hover { color: #79c0ff; }
        .select-filter { background-color: #ffffff; border: 1px solid #cbd5e1; color: #4a5568; border-radius: 6px; padding: 0.5rem; }
        .toggle-btn {
            /* Reduced size and repositioned next to the title */
            width: 40px;
            height: 40px;
            border-radius: 50%;
            display: flex; /* Use flex to enable justify-content and align-items */
            align-items: center;
            justify-content: center;
            font-size: 1.2rem; /* Reduced font size */
            color: white;
            cursor: pointer;
            transition: background-color 0.3s, transform 0.2s;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2); /* Lighter shadow */
            /* Margins will be controlled by the flex container (header-title-group) */
            flex-shrink: 0; /* Prevent button from shrinking */
        }
        .toggle-btn.electricity { background-color: #eab308; } /* Electricity color (yellow) */
        .toggle-btn.water { background-color: #3b82f6; } /* Water color (blue) */
        .toggle-btn:hover { transform: scale(1.05); }

        .header-title-group {
           display: flex;
           align-items: center;
           gap: 10px; /* Space between button and title */
           /* To remove absolute positioning from the button, make the group itself flex */
        }

        .summary-row { /* For consumption summary row (consider splitting right and left) */
            display: flex;
            justify-content: space-between; /* Distribute space between elements */
            align-items: center;
            flex-wrap: wrap; /* Allow elements to wrap to a new line on small screens */
            gap: 1rem; /* Space between elements */
        }

        .summary-item { /* For individual summary item (e.g., total Kahramaa, total Ashghal) */
            flex-grow: 1; /* Allow it to expand */
            min-width: 150px; /* Prevent excessive shrinking on small screens */
            text-align: left; /* Align text to the left */
            display: flex; /* Use flexbox for icon and text block */
            align-items: center; /* Vertically align icon with text block */
        }
        .summary-item .text-block {
           display: flex;
           flex-direction: column;
           text-align: left; /* Ensure text within block is left-aligned */
        }
        .summary-item p {
           text-align: left; /* Align text to the left */
        }
        .summary-total-value {
            font-size: 1.5rem; /* Reduced font size text-2xl to text-xl */
            font-weight: 700; /* font-bold */
            color: #1f2937; /* Dark text color to highlight total */
        }
        .summary-label {
            color: #64748b; /* Medium gray text */
            font-size: 0.875rem; /* text-sm */
        }
        /* Font size for individual meter readings in cards */
        .meter-reading-display {
            font-size: 0.875rem; /* text-sm */
        }
        .meter-consumption-display {
            font-size: 1.25rem; /* text-lg */
        }

        .view-meter-image-btn { /* View image button inside the card */
            color: #58a6ff;
            cursor: pointer;
            margin-left: 0.5rem;
            font-size: 1.1rem;
            transition: color 0.2s;}
        .view-meter-image-btn:hover {
            color: #79c0ff;
        }
        /* Beige colors for Ashghal meters */
        .text-beige-icon { color: #8d6e35; /* Dark brown color for icons */}
        .border-beige-meter-card { border-color: #d4a762; /* Light brown color for borders */}
        .bg-beige-consumption-ashghal { background-color: rgba(212, 167, 98, 0.2); /* Transparent beige for background */}


        /* Print styles */
        @media print {
            body { background-color: white; color: black; }
            .no-print { display: none!important; }
            .card { border: 1px solid #ccc; box-shadow: none; }
            .container { max-width: 100%; }
            h1, h2, h3, p, span, div { color: black!important; }
        }

        /* Mobile specific adjustments */
        @media (max-width: 639px) { /* Tailwind's 'sm' breakpoint */
            .header-title-group {
                flex-direction: column; /* Stack title and toggle button */
                align-items: center;
                text-align: center;
                width: 100%;
                margin-bottom: 1rem; /* Add some space below the title group */
            }

            #main-title {
                font-size: 1.5rem; /* Smaller font size for main title on mobile */
                margin-bottom: 0.5rem; /* Reduce margin */
            }

            .toggle-btn {
                margin-bottom: 0.5rem; /* Add space below toggle button when stacked */
            }

            header.mb-8 {
                flex-direction: column; /* Stack header elements */
                align-items: center;
                gap: 1rem; /* Space between stacked elements */
            }

            header > div {
                width: 100%; /* Make child divs full width */
                justify-content: center; /* Center content */
            }

            .summary-item {
                min-width: unset; /* Remove min-width for better wrapping */
                width: 100%; /* Take full width when stacked */
                justify-content: center; /* Center content in summary items */
                text-align: center;
            }

            .summary-item .text-block {
                align-items: center; /* Center text block content */
                text-align: center;
            }

            .summary-item p {
                text-align: center; /* Center text within summary items */
            }

            .grid-cols-1.md:grid-cols-2.lg:grid-cols-4 {
                grid-template-columns: 1fr; /* Force single column on mobile */
            }

            .meter-reading-display, .meter-consumption-display {
                font-size: 1rem; /* Slightly smaller font for readings on mobile */
            }

            .view-image-btn, .view-meter-image-btn {
                font-size: 1rem; /* Smaller icon size on mobile */
            }

            .nav-btn {
                padding: 0.5rem 0.75rem; /* Smaller padding for nav buttons */
            }

            #current-period-label {
                min-width: 120px; /* Adjust min-width for date label */
                font-size: 0.9rem; /* Smaller font for date label */
            }

            .select-filter {
                width: 100%; /* Full width for dropdown on mobile */
            }
        }
    </style>
</head>
<body class="p-4 sm:p-6 lg:p-8 flex flex-col min-h-screen">
    <div id="printable-area" class="container mx-auto flex-grow">
        <header class="mb-8 flex flex-col sm:flex-row justify-between items-center no-print relative">
            <div class="header-title-group"> <!-- Title and button group -->
                <!-- Toggle button between Electricity and Water -->
                <button id="dashboard-toggle-btn" class="toggle-btn">
                    <span id="toggle-icon-text"></span> <!-- Changed to text instead of i icon -->
                </button>
                <h1 id="main-title" class="text-2xl sm:text-3xl font-bold text-gray-800 mb-0 sm:mb-0"></h1>
            </div>
            <div class="flex items-center gap-4 flex-wrap justify-center">
                <!-- Dark Mode Toggle Button -->
                <button id="dark-mode-toggle" class="p-2 rounded-md nav-btn flex items-center justify-center">
                    <i class="bi bi-moon-fill" id="dark-mode-icon"></i>
                    <!-- Removed the span for dark-mode-text to make the button smaller -->
                </button>
                <button id="print-btn" class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded flex items-center gap-2"><i class="bi bi-printer-fill"></i><span id="print-btn-text"></span></button>
                 <div class="flex items-center gap-3">
                    <button id="prev-btn" class="p-2 rounded-md nav-btn"><i class="bi bi-chevron-left"></i></button>
                    <span id="current-period-label" class="text-lg font-semibold text-gray-800 w-auto min-w-[150px] text-center"></span>
                    <button id="next-btn" class="p-2 rounded-md nav-btn"><i class="bi bi-chevron-right"></i></button>
                </div>
            </div>
        </header>
        <section class="grid grid-cols-1 gap-6 mb-8">
            <div class="card p-5">
                <div id="summary-content" class="flex items-center gap-4 flex-wrap">
                    <!-- This content will be dynamically populated by JavaScript -->
                    <!-- The main summary icon is now placed dynamically inside the summary-item for electricity -->
                    <div id="main-summary-display" class="flex-grow">
                        <h3 id="summary-title" class="text-gray-600 text-md"></h3>
                        <p id="summary-total" class="summary-total-value text-2xl font-bold text-gray-900">--- KWH</p>
                    </div>
                </div>
            </div>
        </section>
        <div id="meters-container" class="space-y-8">
             <div class="loader-container text-center py-10"><div class="loader"></div><p id="loading-text"></p></div>
        </div>
        <div class="card p-6 h-fit mt-8">
            <div class="flex flex-col sm:flex-row justify-between items-center mb-4 gap-2">
                <h3 id="chart-title" class="text-xl font-bold text-gray-800"></h3>
                <select id="meter-chart-filter" class="select-filter no-print"></select>
            </div>
            <div class="relative" style="height: 40vh;"><canvas id="consumptionChart"></canvas></div>
            <div class="flex justify-center items-center gap-4 mt-4 no-print flex-wrap">
                <button id="prev-view-btn" class="p-2 rounded-full nav-btn"><i class="bi bi-arrow-left-circle-fill text-xl"></i></button>
                <span id="view-mode-label" class="text-lg font-semibold text-gray-800 w-48 text-center"></span>
                <button id="next-view-btn" class="p-2 rounded-full nav-btn"><i class="bi bi-arrow-right-circle-fill text-xl"></i></button>
            </div>
        </div>
    </div>
    <footer id="app-footer" class="text-center text-xs text-gray-500 mt-10 pb-4 no-print">
        <span id="footer-text-content"></span>
    </footer>

    <script type="module">
    // Defer execution until the DOM is fully loaded and Firebase is ready.
    document.addEventListener('DOMContentLoaded', async function() {
        console.log("DEBUG: DOMContentLoaded event fired.");

        // Wait for Firebase to be initialized and authenticated
        if (typeof window.authReadyPromise !== 'undefined') {
            await window.authReadyPromise;
            console.log("DEBUG: Firebase authReadyPromise resolved. Firebase services are ready.");
        } else {
            console.error("DEBUG: Firebase authReadyPromise is NOT defined. Firebase initialization might have failed at a very early stage.");
            if (loaderContainer) {
                loaderContainer.innerHTML = `<p class="text-red-500">${translations.en.dbError}</p>`;
            }
            return;
        }

        const db = window.db;
        const auth = window.auth;
        const collection = window.firestoreCollection;
        const onSnapshot = window.firestoreOnSnapshot;

        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        console.log(`DEBUG: Using App ID: ${appId}`);

        // Define meter structure for both electricity and water
        const allMetersData = {
            electricity: {
                dashboardTitle: { en: "Electricity Meters Dashboard" },
                summaryIcon: "bi-lightning-charge-fill text-yellow-400", // This is now a class for the icon, not an ID
                summaryUnit: "KWH",
                collection: `/artifacts/${appId}/public/data/electricityReadings`,
                categories: {
                    lvp: {
                        title: { en: "Low Voltage Panels (LVP)" },
                        sub_categories: {
                            main_panels: {
                                title: { en: "Main Panels" },
                                panels: [
                                    { id: 'LVP-1', location: { en: 'LVP-1 Parking Lights + Sewage Pumps' } },
                                    { id: 'LVP-2A', location: { en: 'LVP-2A B1 lights + AHU + Elevator + Water Heater' } },
                                    { id: 'LVP-3A', location: { en: 'LVP-3A B2 AHU+Lights+Water Heater+Data Center' } }
                                    ,
                                    { id: 'LVP-4', location: { en: 'LVP-4 Chiller 1&2' } },
                                    { id: "LVP-5", location: { en: "LVP-5 Chiller 3&4" } },
                                    // Added the missing electricity meters
                                    { id: "LVP-6", location: { en: "LVP-6 Cooling Tower+Pumps" } },
                                    { id: "LVP-7A", location: { en: "LVP-7A B3 AHU+Lights+B7 lights" } },
                                    { id: "LVP-8A", location: { en: "LVP-8A B5 & B4 loads" } }
                                ]
                            }
                        }
                    }
                } 
            },
            water: {
                dashboardTitle: { en: "Water Meters Dashboard" },
                summaryIcon: "bi-droplet-fill text-blue-400", // This is now a class for the icon, not an ID
                summaryUnit: "M³", // Cubic meter
                collection: `/artifacts/${appId}/public/data/waterReadings`,
                categories: {
                    kahramaa: {
                        title: { en: "Kahramaa Water (Domestic)" },
                        sub_categories: {
                            main_meter: {
                                title: { en: "Main Meter" },
                                panels: [{ id: '1219053', location: { en: 'Main Kahramaa Meter' } }]
                            },
                            sub_meters: {
                                title: { en: "Sub-Meters" },
                                panels: [
                                    { id: 'KP-8530916', location: { en: 'Makeup Water Tank Branch' } },
                                    { id: 'KF-18540109', location: { en: 'Fire Tank + Irrigation Tank Branch' } },
                                    { id: 'GR-01', location: { en: 'Guard Room 01' } },
                                    { id: 'GR-02', location: { en: 'Guard Room 02' } }
                                ]
                            },
                            buildings: {
                                title: { en: "Building Meters" },
                                panels: [
                                    { id: 'B1', location: {en: 'Evaluation Institute' } },
                                    { id: 'B2', location: {en: 'Shared Services Division' } },
                                    { id: 'B3', location: {en: 'Higher Education Institute' } },
                                    { id: 'B4', location: {en: 'Secretary General\'s Office' } },
                                    { id: 'B5', location: {en: 'Education Institute' } }
                                ]
                            },
                            water_fountains_meter: { 
                                title: { en: "Water Features" }, 
                                panels: [{ id: 'WF', location: { en: 'Water Features' } }] // Removed (Previous and Current)
                            }
                        }
                    },
                    ashghal: {
                        title: { en: "Ashghal (TSE) Water" },
                        sub_categories: {
                            main_meter: {
                                title: { en: "Main Meter" },
                                panels: [{ id: '19ACI 005333', location: { en: 'Main TSE Meter' } }]
                            },
                            sub_meters: {
                                title: { en: "Sub-Meters" },
                                panels: [
                                    { id: 'K22223190003505907', location: { en: 'Irrigation Tank Branch (Dm³)' }, conversionFactor: 0.001 },
                                    { id: 'KF-18540133', location: { en: 'RO Plant Branch (Pre-RO)' } },
                                    // Cooling towers meter is re-added here to be displayed, but will not be counted in the total Ashghal sum unless explicitly specified
                                    { id: '15215002323', location: { en: 'Cooling Towers Meter (Post-Makeup Tank)' } }
                                ]
                            }
                        }
                    }
                }
            }
        };

        const translations = { 
            en: { 
                dashboardTitle: { electricity: "Electricity Meters Dashboard", water: "Water Meters Dashboard" },
                summaryTitle: "Total Consumption", chartTitle: "Consumption Trends", 
                prevReading: "Previous", currentReading: "Current", consumption: "Consumption", 
                noData: "No data for this period.", 
                loading: "Connecting...", dbError: "Database connection error", 
                printBtn: "Print", footerText: "Designed & Developed by: Eng. Ali Abdulkarim Elhassan", 
                chartSummary: "Overall Summary", view_daily_7: "Last 7 Days", view_daily_30: "Last 30 Days", 
                view_month: "Month ", view_yearly: "Year ", 
                generating_report: "Generating Report...",
                toggle_to_electricity: "E", 
                toggle_to_water: "W", 
                category_lvp: "Low Voltage Panels (LVP)", 
                category_kahramaa: "Kahramaa Water (Domestic)",
                sub_category_main_meter: "Main Meter",
                sub_category_sub_meters: "Sub-Meters",
                sub_category_buildings: "Building Meters",
                sub_category_water_fountains_meter: "Water Features", // Added for WF
                category_ashghal: "Ashghal (TSE) Water",
                darkMode: "Dark Mode", // New translation
                lightMode: "Light Mode", // New translation
                viewImage: "View Image", // New translation
                pdfDateLabel: "Date", // New translation for PDF report
                pdfMonthLabel: "Month", // New translation for PDF report
                pdfYearLabel: "Year", // New translation for PDF report
                consumptionUnit: "Consumption (Unit)" // New translation for Y-axis label
            }
        };

        const mainTitleEl = document.getElementById('main-title');
        const currentPeriodLabel = document.getElementById('current-period-label');
        const prevBtn = document.getElementById('prev-btn');
        const nextBtn = document.getElementById('next-btn'); 
        const metersContainer = document.getElementById('meters-container');
        const meterChartFilter = document.getElementById('meter-chart-filter');
        const chartTitleEl = document.getElementById('chart-title');
        const viewModeLabel = document.getElementById('view-mode-label');
        const prevViewBtn = document.getElementById('prev-view-btn');
        const nextViewBtn = document.getElementById('next-view-btn');
        const printBtn = document.getElementById('print-btn');
        const loaderContainer = document.querySelector('.loader-container');
        const summaryTotalEl = document.getElementById('summary-total');
        // Removed: const summaryIconEl = document.getElementById('summary-icon'); // No longer needed as global icon
        const dashboardToggleBtn = document.getElementById('dashboard-toggle-btn');
        const toggleIconEl = document.getElementById('toggle-icon-text'); 
        const summaryContentEl = document.getElementById('summary-content'); 
        const footerTextContentEl = document.getElementById('footer-text-content'); // Get the new span for footer text
        const darkModeToggleBtn = document.getElementById('dark-mode-toggle'); // New dark mode button
        const darkModeIcon = document.getElementById('dark-mode-icon'); // New dark mode icon


        let currentLanguage = 'en'; // Set to English
        let currentDashboardType = 'water'; 
        let selectedDate = new Date(); 
        let allReadings = {}; 
        let consumptionChart = null;
        let unsubscribeFirestore = null; 

        const viewModes = ['daily_7', 'daily_30', 'monthly', 'yearly'];
        let currentViewModeIndex = 0; 

        // Function to apply dark mode preference from localStorage
        function applyDarkModePreference() {
            const isDarkMode = localStorage.getItem('darkMode') === 'true';
            if (isDarkMode) {
                document.body.classList.add('dark-mode');
                darkModeIcon.className = 'bi bi-sun-fill';
            } else {
                document.body.classList.remove('dark-mode');
                darkModeIcon.className = 'bi bi-moon-fill';
            }
        }

        // Dark Mode Toggle Event Listener
        darkModeToggleBtn.addEventListener('click', () => {
            const isDarkMode = document.body.classList.contains('dark-mode');
            localStorage.setItem('darkMode', !isDarkMode);
            applyDarkModePreference();
            updateChart(); // Re-render chart to apply new colors
        });

        // Apply dark mode preference on initial load
        applyDarkModePreference();

        // Helper function to format date to YYYY-MM-DD string
        function formatDateToYYYYMMDD(date) {
            const year = date.getFullYear();
            const month = (date.getMonth() + 1).toString().padStart(2, '0');
            const day = date.getDate().toString().padStart(2, '0');
            return `${year}-${month}-${day}`;
        }

        // Helper function to get a previous date
        function getPreviousDate(date, days = 1) { 
            const d = new Date(date); 
            d.setDate(d.getDate() - days); 
            return d; 
        }

        // Helper function to get meter data (value and imageUrl) for a specific meter and date
        function getMeterDataForDate(meterId, date) {
            const dateStr = formatDateToYYYYMMDD(date);

            // Special logic for 'WF' meter (Water Features)
            if (meterId === 'WF') {
                const mainKahramaaMeterId = '1219053';
                const subAndBuildingMeters = [
                    'KP-8530916', 'KF-18540109', 'GR-01', 'GR-02', 
                    'B1', 'B2', 'B3', 'B4', 'B5', 
                    '15215002323'
                ];

                const mainReading = allReadings?.[dateStr]?.[mainKahramaaMeterId]?.value;
                let sumSubAndBuildingReadings = 0;

                subAndBuildingMeters.forEach(subMeterId => {
                    const subReading = allReadings?.[dateStr]?.[subMeterId]?.value;
                    if (subReading !== undefined) {
                        sumSubAndBuildingReadings += parseFloat(subReading);
                    }
                });

                if (mainReading !== undefined) {
                    const calculatedWF = parseFloat(mainReading) - sumSubAndBuildingReadings;
                    return { value: calculatedWF, imageUrl: null }; // WF has no direct image
                }
                return { value: undefined, imageUrl: null };
            }
            // Normal logic for other meters: return the whole data object
            return allReadings?.[dateStr]?.[meterId]; 
        }

        // Helper function to get a specific meter's configuration from allMetersData
        function getMeterConfig(meterId) {
            const currentMetersConfig = allMetersData[currentDashboardType];
            for (const categoryKey in currentMetersConfig.categories) {
                const category = currentMetersConfig.categories[categoryKey];
                for (const subCategoryKey in category.sub_categories) {
                    const foundMeter = category.sub_categories[subCategoryKey].panels.find(m => m.id === meterId);
                    if (foundMeter) return foundMeter;
                }
            }
            return null;
        }

        // Updates all dashboard components
        function renderAll() { 
            console.log("DEBUG: renderAll function started.");
            updateDashboard(); 
            updateChart();
            updatePeriodLabel();
            updateViewModeLabel();
            updateToggleBtn();
            updateFooterText(); // Update footer text on every render
            console.log("DEBUG: renderAll function finished.");
        }

        // Set application language and update UI elements
        function setLanguage(lang) {
            console.log(`DEBUG: Setting language to: ${lang}`);
            currentLanguage = lang;
            document.documentElement.lang = lang;
            // Set text direction based on language
            document.documentElement.dir = 'ltr'; // Always LTR for English

            // Set chevron icons for navigation buttons
            prevBtn.innerHTML = `<i class="bi bi-chevron-left"></i>`; 
            nextBtn.innerHTML = `<i class="bi bi-chevron-right"></i>`; 

            document.body.className = `p-4 sm:p-6 lg:p-8 flex flex-col min-h-screen lang-${lang}`;
            document.getElementById('summary-title').textContent = translations[lang].summaryTitle;
            document.getElementById('print-btn-text').textContent = translations[lang].printBtn;
            const loadingTextEl = document.getElementById('loading-text');
            if (loadingTextEl) loadingTextEl.textContent = translations[lang].loading;
            populateMeterDropdown(); // Re-populate dropdown on language change
            renderAll();
            console.log(`DEBUG: Language set to ${lang}.`);
        }

        // Update the dashboard toggle button (Electricity/Water)
        function updateToggleBtn() {
            console.log("DEBUG: Updating toggle button.");
            let buttonText = "";
            let buttonTitle = "";
            let buttonBgColor = "";

            if (currentDashboardType === 'electricity') {
                buttonText = translations[currentLanguage].toggle_to_water;
                buttonTitle = translations[currentLanguage].toggle_to_water;
                buttonBgColor = '#3b82f6';
            } else {
                buttonText = translations[currentLanguage].toggle_to_electricity;
                buttonTitle = translations[currentLanguage].toggle_to_electricity;
                buttonBgColor = '#eab308';
            }

            toggleIconEl.textContent = buttonText; 
            dashboardToggleBtn.title = buttonTitle;
            dashboardToggleBtn.style.backgroundColor = buttonBgColor;
            dashboardToggleBtn.style.textAlign = 'center';
            console.log("DEBUG: Toggle button updated.");
        }

        // Function to update the footer text
        function updateFooterText() {
            if (footerTextContentEl) {
                footerTextContentEl.textContent = translations[currentLanguage].footerText;
            }
        }

        // Populate the meter dropdown list for charts
        function populateMeterDropdown() {
            console.log("DEBUG: Populating meter dropdown.");
            // Always add the "Overall Summary" option first
            meterChartFilter.innerHTML = `<option value="summary">${translations[currentLanguage].chartSummary}</option>`;

            const currentMetersConfig = allMetersData[currentDashboardType];

            for (const categoryKey in currentMetersConfig.categories) {
                const category = currentMetersConfig.categories[categoryKey];
                const categoryTitle = translations[currentLanguage][`category_${categoryKey}`] || category.title[currentLanguage];

                // Create a main optgroup for the category (e.g., Low Voltage Panels (LVP))
                const categoryOptgroup = document.createElement('optgroup');
                categoryOptgroup.label = categoryTitle;

                for (const subCategoryKey in category.sub_categories) {
                    const subCategory = category.sub_categories[subCategoryKey];

                    subCategory.panels.forEach(meter => {
                        const option = document.createElement('option');
                        option.value = meter.id;
                        // For WF, the location text now simply says "Water Features"
                        option.textContent = `${meter.id} (${meter.location[currentLanguage]})`; 
                        categoryOptgroup.appendChild(option);
                    });
                }

                // Only append the category optgroup if it contains meters
                if (categoryOptgroup.children.length > 0) {
                    meterChartFilter.appendChild(categoryOptgroup);
                }
            }
            console.log("DEBUG: Meter dropdown populated.");
        }


        // Update the time period label (Date/Month/Year) in the top navigation bar
        function updatePeriodLabel() {
            console.log(`DEBUG: Updating period label. Current index: ${currentViewModeIndex}`);
            const currentViewMode = viewModes[currentViewModeIndex];
            let labelText = '';

            // Determine locale based on current language for date formatting
            const locale = currentLanguage === 'ar' ? 'ar-EG' : 'en-GB'; // Changed to 'en-GB' for Day Month Year format

            // Options for displaying day, month, and year
            const dateOptions = { day: '2-digit', month: 'long', year: 'numeric' };

            switch (currentViewMode) {
                case 'daily_7':
                case 'daily_30':
                    labelText = new Intl.DateTimeFormat(locale, dateOptions).format(selectedDate);
                    break;
                case 'monthly':
                    // For monthly view, only show month and year
                    labelText = new Intl.DateTimeFormat(locale, { month: 'long', year: 'numeric' }).format(selectedDate);
                    break;
                case 'yearly':
                    labelText = `${selectedDate.getFullYear()}`;
                    break;
            }
            currentPeriodLabel.textContent = labelText;

            // Adjust width of the label to accommodate longer month names
            currentPeriodLabel.classList.remove('w-44'); // Remove previous fixed width
            currentPeriodLabel.classList.add('w-auto', 'min-w-[150px]'); // Use auto width with a minimum to allow expansion

            const today = new Date(); today.setHours(0,0,0,0);
            const earliestYear = 2021;

            nextBtn.disabled = false;

            switch(currentViewMode) {
                case 'daily_7':
                case 'daily_30':
                    nextBtn.disabled = selectedDate.toDateString() === today.toDateString() || selectedDate > today;
                    break;
                case 'monthly':
                    const currentMonthStart = new Date(today.getFullYear(), today.getMonth(), 1);
                    const selectedMonthStart = new Date(selectedDate.getFullYear(), selectedDate.getMonth(), 1);
                    nextBtn.disabled = selectedMonthStart >= currentMonthStart;
                    prevBtn.disabled = selectedDate.getFullYear() < earliestYear;
                    break;
                case 'yearly':
                    nextBtn.disabled = selectedDate.getFullYear() >= today.getFullYear();
                    prevBtn.disabled = selectedDate.getFullYear() <= earliestYear;
                    break;
            }
            console.log("DEBUG: Period label updated.");
        }

        // Updates the main dashboard display with meter readings and consumption
        function updateDashboard() {
            console.log("DEBUG: updateDashboard function started.");
            if(!selectedDate) {
                console.log("DEBUG: selectedDate is null/undefined in updateDashboard. Exiting.");
                return;
            }

            mainTitleEl.textContent = allMetersData[currentDashboardType].dashboardTitle[currentLanguage];
            // Removed: summaryIconEl.className = `${allMetersData[currentDashboardType].summaryIcon} text-3xl`; // No longer a global icon

            metersContainer.innerHTML = '';

            const currentMetersConfig = allMetersData[currentDashboardType];
            const currentViewMode = viewModes[currentViewModeIndex];
            const isDailyView = ['daily_7', 'daily_30'].includes(currentViewMode);

            let totalConsumptionOverall = 0;

            let summaryHtmlContent = ''; // Start with empty content

            if (currentDashboardType === 'water') {
                let totalKahramaaConsumption = 0;
                let totalAshghalConsumption = 0;

                const { consumption: kahramaaMainComplexCons } = calculateAggregatedConsumptionForCategory('1219053', selectedDate, currentViewMode);
                totalKahramaaConsumption = kahramaaMainComplexCons;

                // --- Start: Explicitly calculate Ashghal total from Irrigation and RO Plant ONLY ---
                const ashghalIrrigationTankId = 'K22223190003505907';
                const ashghalROPlantId = 'KF-18540133';

                let tempAshghalConsumption = 0;
                const year = selectedDate.getFullYear();
                const month = selectedDate.getMonth();
                let startDay, endDay;

                if (currentViewMode === 'daily_7' || currentViewMode === 'daily_30') {
                    // For daily view, sum daily consumption directly with conversion
                    tempAshghalConsumption += getDailyConsumptionForAnyMeter(ashghalIrrigationTankId, selectedDate, true); // Sum in M3
                    tempAshghalConsumption += getDailyConsumptionForAnyMeter(ashghalROPlantId, selectedDate, true);   // Sum in M3
                } else if (currentViewMode === 'monthly') {
                    startDay = new Date(year, month, 1);
                    endDay = new Date(year, month + 1, 0); // Last day of the month
                    let tempDate = new Date(startDay);
                    while (tempDate <= endDay) {
                        tempAshghalConsumption += getDailyConsumptionForAnyMeter(ashghalIrrigationTankId, tempDate, true); // Sum in M3
                        tempAshghalConsumption += getDailyConsumptionForAnyMeter(ashghalROPlantId, tempDate, true);   // Sum in M3
                        tempDate.setDate(tempDate.getDate() + 1);
                    }
                } else if (currentViewMode === 'yearly') {
                    startDay = new Date(year, 0, 1);
                    endDay = new Date(year, 11, 31); // Last day of the year
                    let tempDate = new Date(startDay);
                    while (tempDate <= endDay) {
                        tempAshghalConsumption += getDailyConsumptionForAnyMeter(ashghalIrrigationTankId, tempDate, true); // Sum in M3
                        tempAshghalConsumption += getDailyConsumptionForAnyMeter(ashghalROPlantId, tempDate, true);   // Sum in M3
                        tempDate.setDate(tempDate.getDate() + 1);
                    }
                }
                totalAshghalConsumption = tempAshghalConsumption;
                console.log("DEBUG: Calculated totalAshghalConsumption:", totalAshghalConsumption); // Debug log
                // --- End: Explicitly calculate Ashghal total ---

                totalConsumptionOverall = totalKahramaaConsumption + totalAshghalConsumption;

                summaryHtmlContent = `
                    <div class="flex-grow grid grid-cols-1 sm:grid-cols-2 gap-4 w-full">
                        <div class="summary-item">
                            <i class="bi bi-droplet-fill text-2xl text-blue-400 mr-2"></i> <!-- Kahramaa Icon -->
                            <div class="text-block">
                                <h3 class="text-gray-600 text-md">${translations[currentLanguage].category_kahramaa}</h3>
                                <p class="text-gray-600 text-md">${translations[currentLanguage].summaryTitle}: <span class="summary-total-value">${totalKahramaaConsumption.toFixed(2)} ${currentMetersConfig.summaryUnit}</span></p>
                            </div>
                        </div>
                        <div class="summary-item">
                            <i class="bi bi-recycle text-2xl text-beige-icon mr-2"></i>
                            <div class="text-block">
                                <h3 class="text-gray-600 text-md">${translations[currentLanguage].category_ashghal}</h3> 
                                <p class="text-gray-600 text-md">${translations[currentLanguage].summaryTitle}: <span class="summary-total-value">${totalAshghalConsumption.toFixed(2)} ${currentMetersConfig.summaryUnit}</span></p>
                            </div>
                        </div>
                    </div>
                `;
            } else { // Electricity Dashboard
                const lvpCategories = currentMetersConfig.categories.lvp;
                const lvpMeters = [];
                for(const subCatKey in lvpCategories.sub_categories){
                    lvpMeters.push(...lvpCategories.sub_categories[subCatKey].panels);
                }

                lvpMeters.forEach(meter => {
                    const { consumption: meterCons } = calculateAggregatedConsumption(meter.id); 
                    totalConsumptionOverall += meterCons;
                });

                summaryHtmlContent = `
                    <i class="bi bi-lightning-charge-fill text-3xl text-yellow-400 mr-2"></i> <!-- Electricity Icon -->
                    <div id="main-summary-display" class="flex-grow">
                        <h3 class="text-gray-600 text-md">${translations[currentLanguage].summaryTitle}</h3>
                        <p class="summary-total-value">${totalConsumptionOverall.toFixed(2)} ${currentMetersConfig.summaryUnit}</p>
                    </div>
                `;
            }
            summaryContentEl.innerHTML = summaryHtmlContent;


            let fullHtmlContent = '';

            for (const categoryKey in currentMetersConfig.categories) {
                const category = currentMetersConfig.categories[categoryKey];
                const categoryTitle = translations[currentLanguage][`category_${categoryKey}`] || category.title[currentLanguage];

                fullHtmlContent += `
                    <div class="space-y-6">
                        <h2 class="text-2xl font-bold text-gray-800 mb-4 pb-2 border-b-2 ${currentDashboardType === 'electricity' ? 'border-yellow-500' : 'border-blue-500'}">
                            ${categoryTitle}
                        </h2>
                `;

                for (const subCategoryKey in category.sub_categories) {
                    const subCategory = category.sub_categories[subCategoryKey];
                    const subCategoryTitle = translations[currentLanguage][`sub_category_${subCategoryKey}`] || subCategory.title[currentLanguage];

                    fullHtmlContent += `
                        <h3 class="text-lg font-semibold text-gray-700 mb-3">${subCategoryTitle}</h3>
                        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                    `;

                    subCategory.panels.forEach(meter => {
                        let cardContentHtml = ''; 
                        let consumptionTextColor = currentDashboardType === 'electricity' ? 'text-yellow-700' : 'text-blue-700';
                        let consumptionBgColor = currentDashboardType === 'electricity' ? 'bg-yellow-100' : 'bg-blue-100';

                        // Check if the current meter is part of Ashghal's sub_meters (specifically excluding cooling towers from ashghal's count, but not from its card display)
                        if (currentDashboardType === 'water' && Object.values(allMetersData.water.categories.ashghal.sub_categories).some(sub => sub.panels.some(p => p.id === meter.id))) {
                            consumptionBgColor = 'bg-beige-consumption-ashghal';
                            consumptionTextColor = 'text-beige-icon'; 
                        }

                        // Special handling for WF meter: always show only consumption
                        if (meter.id === 'WF') {
                             const currentWFData = getMeterDataForDate('WF', selectedDate);
                             const prevWFData = getMeterDataForDate('WF', getPreviousDate(selectedDate));
                             let consumptionDisplay = "N/A";
                             if (currentWFData.value !== undefined && prevWFData.value !== undefined) {
                                 consumptionDisplay = (currentWFData.value - prevWFData.value).toFixed(2);
                             }
                             cardContentHtml = `
                                <div class="text-center mt-2 border-t border-gray-300 pt-3">
                                    <p class="text-xs ${consumptionTextColor}">${translations[currentLanguage].consumption}</p>
                                    <p class="text-3xl font-bold ${consumptionTextColor}">${consumptionDisplay} ${currentMetersConfig.summaryUnit}</p>
                                </div>
                            `;
                        } else if (isDailyView) { 
                            const currentMeterData = getMeterDataForDate(meter.id, selectedDate);
                            const prevMeterData = getMeterDataForDate(meter.id, getPreviousDate(selectedDate));

                            let currentReadingDisplay = currentMeterData?.value !== undefined ? currentMeterData.value : 'N/A';
                            let prevReadingDisplay = prevMeterData?.value !== undefined ? prevMeterData.value : 'N/A';
                            let imageUrlForMeter = currentMeterData?.imageUrl || null;
                            let prevImageUrlForMeter = prevMeterData?.imageUrl || null;

                            let consumptionDisplay = getDailyConsumptionForAnyMeter(meter.id, selectedDate, true).toFixed(2);

                            const viewImageButtonHtml = imageUrlForMeter ? 
                                `<button type="button" class="view-meter-image-btn" onclick="window.open('${imageUrlForMeter}', '_blank')" title="${translations[currentLanguage].viewImage}"><i class="bi bi-eye-fill"></i></button>` : '';
                            const prevViewImageButtonHtml = prevImageUrlForMeter ? 
                                `<button type="button" class="view-meter-image-btn" onclick="window.open('${prevImageUrlForMeter}', '_blank')" title="${translations[currentLanguage].viewImage} ${translations[currentLanguage].prevReading}"><i class="bi bi-eye-fill"></i></button>` : '';

                            cardContentHtml = `
                                <div class="grid grid-cols-3 gap-2 text-center mt-2 border-t border-gray-300 pt-3">
                                    <div class="flex flex-col items-center">
                                        <p class="text-xs text-gray-500">${translations[currentLanguage].prevReading}</p>
                                        <p class="meter-reading-display font-semibold text-gray-800">${prevReadingDisplay}</p>
                                        ${prevViewImageButtonHtml}
                                    </div>
                                    <div class="flex flex-col items-center">
                                        <p class="text-xs text-gray-500">${translations[currentLanguage].currentReading}</p>
                                        <p class="meter-reading-display font-semibold text-gray-800">${currentReadingDisplay}</p>
                                        ${viewImageButtonHtml}
                                    </div>
                                    <div class="${consumptionBgColor} rounded-md p-1">
                                        <p class="text-xs ${consumptionTextColor}">${translations[currentLanguage].consumption}</p>
                                        <p class="meter-consumption-display font-bold ${consumptionTextColor}">${consumptionDisplay} ${currentMetersConfig.summaryUnit}</p>
                                    </div>
                                </div>
                            `;
                        } else { // Monthly or Yearly view for individual meters (non-WF)
                            const { consumption: aggConsumption } = calculateAggregatedConsumptionForCategory(meter.id, selectedDate, currentViewMode);
                            let consumptionDisplay = aggConsumption.toLocaleString(undefined, {maximumFractionDigits: 0}); 

                            let unitDisplay = currentMetersConfig.summaryUnit;
                            if (meter.id === 'K22223190003505907' && !isDailyView) {
                                unitDisplay = 'Dm³'; // Display Dm³ for irrigation tank in monthly/yearly views
                            }
                            cardContentHtml = `
                                <div class="text-center mt-2 border-t border-gray-300 pt-3">
                                    <p class="text-xs ${consumptionTextColor}">${translations[currentLanguage].consumption}</p>
                                    <p class="meter-consumption-display font-bold ${consumptionTextColor}">${consumptionDisplay} ${unitDisplay}</p>
                                </div>
                            `;
                        }

                        fullHtmlContent += `
                            <div class="card p-4 flex flex-col justify-between ${
                                currentDashboardType === 'water' && (
                                    Object.values(allMetersData.water.categories.kahramaa.sub_categories).some(sub => sub.panels.some(p => p.id === meter.id))
                                ) ? 'border-blue-500' : 
                                currentDashboardType === 'water' && (
                                    Object.values(allMetersData.water.categories.ashghal.sub_categories).some(sub => sub.panels.some(p => p.id === meter.id))
                                ) ? 'border-beige-meter-card' : 
                                currentDashboardType === 'electricity' ? 'border-yellow-500' : ''
                            }">
                                <div>
                                    <p class="font-bold text-lg text-gray-800">${meter.id}</p>
                                    <p class="text-sm text-gray-600 mb-3">${meter.location[currentLanguage]}</p>
                                </div>
                                ${cardContentHtml}
                            </div>
                        `;
                    });
                    fullHtmlContent += '</div>';
                }
                fullHtmlContent += '</div>';

                if (categoryKey !== Object.keys(currentMetersConfig.categories).pop()) {
                    fullHtmlContent += `<hr class="my-8 border-gray-300">`;
                }
            }

            metersContainer.innerHTML = fullHtmlContent; 

            summaryTotalEl.textContent = `${totalConsumptionOverall.toFixed(2)} ${currentMetersConfig.summaryUnit}`;
            console.log("DEBUG: updateDashboard function finished. Total Overall Consumption:", totalConsumptionOverall);
        }

        // Helper function to get daily consumption for any meter.
        // 'applyConversion' parameter controls if the meter's conversionFactor is applied.
        function getDailyConsumptionForAnyMeter(mId, date, applyConversion = true) {
            if (mId === 'WF') {
                // WF logic remains as it is, it calculates difference directly
                const currentWFValue = getMeterDataForDate('WF', date).value;
                const prevWFValue = getMeterDataForDate('WF', getPreviousDate(date)).value;
                if (currentWFValue !== undefined && prevWFValue !== undefined) {
                    const dailyCons = currentWFValue - prevWFValue;
                    return dailyCons > 0 ? dailyCons : 0;
                }
                return 0;
            } else {
                const cReading = getMeterDataForDate(mId, date)?.value;
                const pReading = getMeterDataForDate(mId, getPreviousDate(date))?.value;

                if (cReading === undefined || pReading === undefined) {
                    return 0;
                }

                let dailyCons = parseFloat(cReading) - parseFloat(pReading);

                // Apply conversion factor only if requested AND the meter actually has one defined
                if (applyConversion) {
                    const mConfig = getMeterConfig(mId);
                    if (mConfig && mConfig.conversionFactor !== undefined) {
                        dailyCons *= mConfig.conversionFactor;
                    }
                }

                return dailyCons > 0 ? dailyCons : 0;
            }
        }

        function calculateAggregatedConsumptionForCategory(meterId, dateContext, viewMode) {
            let consumption = 0;
            const currentMetersConfig = allMetersData[currentDashboardType];

            // Identify special meters
            const ashghalIrrigationTankId = 'K22223190003505907';
            const ashghalROPlantId = 'KF-18540133';
            const ashghalMainMeterId = '19ACI 005333';


            // If it's a daily view, calculate based on current day vs. previous day (always apply conversion for display)
            if (viewMode === 'daily_7' || viewMode === 'daily_30') {
                if (currentDashboardType === 'water' && meterId === ashghalMainMeterId) {
                    // For Ashghal summary in daily view, sum up ONLY Irrigation Tank and RO Plant with conversion to M3
                    const irrigationCons = getDailyConsumptionForAnyMeter(ashghalIrrigationTankId, dateContext, true); // Apply conversion
                    const roPlantCons = getDailyConsumptionForAnyMeter(ashghalROPlantId, dateContext, true);   // Apply conversion
                    consumption = irrigationCons + roPlantCons;
                } else {
                    // For any individual meter (including WF, Kahramaa, Electricity, and any other individual sub-meters), get daily consumption with conversion
                    consumption = getDailyConsumptionForAnyMeter(meterId, dateContext, true); // Apply conversion
                }
            } 
            // If it's monthly or yearly view, iterate through days and sum daily consumptions
            else { 
                const year = dateContext.getFullYear();
                const month = dateContext.getMonth();

                let startDay, endDay;
                if (viewMode === 'monthly') {
                    startDay = new Date(year, month, 1);
                    endDay = new Date(year, month + 1, 0);
                } else if (viewMode === 'yearly') {
                    startDay = new Date(year, 0, 1);
                    endDay = new Date(year, 11, 31);
                }

                let tempDate = new Date(startDay);
                while (tempDate <= endDay) {
                    if (currentDashboardType === 'water' && meterId === ashghalMainMeterId) {
                        // For Ashghal summary (monthly/yearly), sum ONLY Irrigation and RO Plant.
                        // Apply conversion for irrigation to ensure overall Ashghal sum is in M3.
                        consumption += getDailyConsumptionForAnyMeter(ashghalIrrigationTankId, tempDate, true); // Sum in M3
                        consumption += getDailyConsumptionForAnyMeter(ashghalROPlantId, tempDate, true);   // Sum in M3
                        tempDate.setDate(tempDate.getDate() + 1);
                    } else if (currentDashboardType === 'water' && meterId === ashghalIrrigationTankId) {
                        // For *individual* Irrigation Tank meter (e.g., in chart filter), user wants Dm3 sum
                        consumption += getDailyConsumptionForAnyMeter(ashghalIrrigationTankId, tempDate, false); // DO NOT apply conversion (sum raw dm3)
                    } else if (currentDashboardType === 'water' && meterId === ashghalROPlantId) {
                        // For *individual* RO Plant meter (e.g., in chart filter), sum M3
                        consumption += getDailyConsumptionForAnyMeter(ashghalROPlantId, tempDate, true); // Apply conversion (its factor is 1, so no change)
                    }
                    else {
                        // For all other meters (Kahramaa, Electricity, WF, and any other individual sub-meters), apply conversion
                        consumption += getDailyConsumptionForAnyMeter(meterId, tempDate, true); // Apply conversion
                    }
                    tempDate.setDate(tempDate.getDate() + 1);
                }
            }
            return { consumption };
        }

        // This function is effectively replaced by calculateAggregatedConsumptionForCategory
        // and is now a helper used within it for daily calculations.
        // Keeping it for clarity, but its direct use for monthly/yearly aggregates is minimalized.
        function calculateAggregatedConsumption(meterId, specificViewMode = null) { 
            // This function now primarily acts as a wrapper to call the more robust
            // calculateAggregatedConsumptionForCategory for consistency.
            // When specificViewMode is null, it uses currentViewModeIndex.
            return calculateAggregatedConsumptionForCategory(meterId, selectedDate, specificViewMode || viewModes[currentViewModeIndex]);
        }

        function getAllMetersFlatInCurrentConfig() {
            let meters = [];
            const currentMetersConfig = allMetersData[currentDashboardType];
            for (const categoryKey in currentMetersConfig.categories) {
                const category = currentMetersConfig.categories[categoryKey];
                for (const subCategoryKey in category.sub_categories) {
                    meters = meters.concat(category.sub_categories[subCategoryKey].panels);
                }
            }
            return meters;
        }


        function generateDailyData(numDays) {
            console.log(`DEBUG: generateDailyData for ${numDays} days.`);
            const selectedMeterId = meterChartFilter.value;
            const labels = []; 
            let dataPointsKahramaa = []; 
            let dataPointsAshghal = []; 
            let dataPointsSingleMeter = []; 
            let datasets = []; 

            const currentMetersConfig = allMetersData[currentDashboardType];
            const allDefinedMeters = getAllMetersFlatInCurrentConfig();
            const ashghalIrrigationTankId = 'K22223190003505907';
            const ashghalROPlantId = 'KF-18540133';
            const ashghalMainMeterId = '19ACI 005333';


            for (let i = numDays - 1; i >= 0; i--) {
                const loopDate = new Date(selectedDate); 
                loopDate.setDate(selectedDate.getDate() - i);

                labels.push(new Intl.DateTimeFormat(currentLanguage === 'ar' ? 'ar-EG' : 'en-US', { month: 'short', day: 'numeric' }).format(loopDate));

                if (selectedMeterId === 'summary') {
                    if (currentDashboardType === 'water') {
                        // Use the new aggregated function with daily_7 to get daily summary for Kahramaa
                        const { consumption: kahramaaTotal } = calculateAggregatedConsumptionForCategory('1219053', loopDate, 'daily_7'); 
                        // For Ashghal total, sum only Irrigation and RO Plant for daily data
                        const irrigationCons = getDailyConsumptionForAnyMeter(ashghalIrrigationTankId, loopDate, true);
                        const roPlantCons = getDailyConsumptionForAnyMeter(ashghalROPlantId, loopDate, true);
                        const ashghalTotal = irrigationCons + roPlantCons; // Sum only these two for Ashghal
                        dataPointsKahramaa.push(parseFloat(kahramaaTotal.toFixed(2)));
                        dataPointsAshghal.push(parseFloat(ashghalTotal.toFixed(2)));
                    } else { 
                        let dailyConsumptionForElectricitySummary = 0;
                        allDefinedMeters.forEach(meterEntry => { 
                            // Direct daily calculation for electricity individual meters
                            dailyConsumptionForElectricitySummary += calculateAggregatedConsumptionForCategory(meterEntry.id, loopDate, 'daily_7').consumption;
                        });
                        dataPointsSingleMeter.push(parseFloat(dailyConsumptionForElectricitySummary.toFixed(2)));
                    }
                } else { 
                    // For individual meter, use the new aggregated function
                    const { consumption: meterCons } = calculateAggregatedConsumptionForCategory(selectedMeterId, loopDate, 'daily_7'); 
                    dataPointsSingleMeter.push(parseFloat(meterCons.toFixed(2))); 
                }
            }

            if (selectedMeterId === 'summary' && currentDashboardType === 'water') {
                datasets.push({
                    label: `${translations[currentLanguage].category_kahramaa} (${currentMetersConfig.summaryUnit})`,
                    data: dataPointsKahramaa,
                    borderColor: '#3b82f6', 
                    backgroundColor: 'rgba(59, 130, 246, 0.2)',
                    fill: true,
                    tension: 0.3
                });
                datasets.push({
                    label: `${translations[currentLanguage].category_ashghal} (${currentMetersConfig.summaryUnit})`,
                    data: dataPointsAshghal,
                    borderColor: '#d4a762', /* Light brown color */
                    backgroundColor: 'rgba(212, 167, 98, 0.2)',
                    fill: true,
                    tension: 0.3
                });
            } else {
                let label = selectedMeterId === 'summary' ? translations[currentLanguage].summaryTitle : selectedMeterId;
                let unit = currentMetersConfig.summaryUnit; // Default unit for electricity and most water meters
                let borderColor = currentDashboardType === 'electricity' ? '#eab308' : '#3b82f6';
                let backgroundColor = currentDashboardType === 'electricity' ? 'rgba(234, 179, 8, 0.2)' : 'rgba(59, 130, 246, 0.2)';

                // Check if the selected meter is an Ashghal meter and apply beige color
                if (currentDashboardType === 'water' && selectedMeterId !== 'summary' && 
                    (selectedMeterId === ashghalIrrigationTankId || selectedMeterId === ashghalROPlantId || selectedMeterId === ashghalMainMeterId || selectedMeterId === '15215002323')) {
                    borderColor = '#d4a762';
                    backgroundColor = 'rgba(212, 167, 98, 0.2)';
                }

                if (selectedMeterId !== 'summary') { 
                    label = `${selectedMeterId} (${getMeterConfig(selectedMeterId).location[currentLanguage]})`;
                }

                datasets.push({ label: `${label} (${unit})`, data: dataPointsSingleMeter, borderColor: borderColor, backgroundColor: backgroundColor, fill: true, tension: 0.3 });
            }

            return { labels, datasets };
        }

        function generateMonthlyData(numMonths) {
            console.log(`DEBUG: generateMonthlyData for ${numMonths} months.`);
            const selectedMeterId = meterChartFilter.value;
            const labels = []; 
            let dataPointsKahramaa = []; 
            let dataPointsAshghal = []; 
            let dataPointsSingleMeter = []; 
            let datasets = []; 

            const currentMetersConfig = allMetersData[currentDashboardType];
            const allDefinedMeters = getAllMetersFlatInCurrentConfig();
            const ashghalIrrigationTankId = 'K22223190003505907';
            const ashghalROPlantId = 'KF-18540133';
            const ashghalMainMeterId = '19ACI 005333';


            for (let i = numMonths - 1; i >= 0; i--) {
                const loopDate = new Date(selectedDate.getFullYear(), selectedDate.getMonth() - i, 1); // First day of the month

                labels.push(new Intl.DateTimeFormat(currentLanguage === 'ar' ? 'ar-EG' : 'en-US', { month: 'short', day: 'numeric' }).format(loopDate));

                if (selectedMeterId === 'summary') {
                    if (currentDashboardType === 'water') {
                        // Calculate aggregated monthly consumption for Kahramaa main meter
                        const { consumption: monthlyKahramaaTotal } = calculateAggregatedConsumptionForCategory('1219053', loopDate, 'monthly'); 
                        dataPointsKahramaa.push(parseFloat(monthlyKahramaaTotal.toFixed(2)));

                        // Calculate aggregated monthly consumption for Ashghal total: sum ONLY Irrigation and RO Plant.
                        let monthlyAshghalTotal = 0;
                        const year = loopDate.getFullYear();
                        const month = loopDate.getMonth();
                        const startDay = new Date(year, month, 1);
                        const endDay = new Date(year, month + 1, 0);
                        let tempDate = new Date(startDay);
                        while (tempDate <= endDay) {
                            monthlyAshghalTotal += getDailyConsumptionForAnyMeter(ashghalIrrigationTankId, tempDate, true); // Sum in M3
                            monthlyAshghalTotal += getDailyConsumptionForAnyMeter(ashghalROPlantId, tempDate, true);   // Sum in M3
                            tempDate.setDate(tempDate.getDate() + 1);
                        }
                        dataPointsAshghal.push(parseFloat(monthlyAshghalTotal.toFixed(2)));

                    } else if (currentDashboardType === 'electricity') {
                        let monthlyTotalConsumption = 0;
                        allDefinedMeters.forEach(meterEntry => {
                            // Sum monthly consumption for each electricity meter
                            monthlyTotalConsumption += calculateAggregatedConsumptionForCategory(meterEntry.id, loopDate, 'monthly').consumption;
                        });
                        dataPointsSingleMeter.push(parseFloat(monthlyTotalConsumption.toFixed(2)));
                    }
                } else { 
                    // For individual meter, calculate its monthly consumption
                    const { consumption: monthlyTotalConsumption } = calculateAggregatedConsumptionForCategory(selectedMeterId, loopDate, 'monthly'); 
                    dataPointsSingleMeter.push(parseFloat(monthlyTotalConsumption.toFixed(2)));
                }
            }

            if (selectedMeterId === 'summary' && currentDashboardType === 'water') {
                datasets.push({
                    label: `${translations[currentLanguage].category_kahramaa} (${currentMetersConfig.summaryUnit})`,
                    data: dataPointsKahramaa,
                    borderColor: '#3b82f6', 
                    backgroundColor: 'rgba(59, 130, 246, 0.2)',
                    fill: true,
                    tension: 0.3
                });
                datasets.push({
                    label: `${translations[currentLanguage].category_ashghal} (${currentMetersConfig.summaryUnit})`,
                    data: dataPointsAshghal,
                    borderColor: '#d4a762', /* Light brown color */
                    backgroundColor: 'rgba(212, 167, 98, 0.2)',
                    fill: true,
                    tension: 0.3
                });
            } else {
                let label = selectedMeterId === 'summary' ? translations[currentLanguage].summaryTitle : selectedMeterId;
                let unit = currentMetersConfig.summaryUnit; // Default unit
                let borderColor = currentDashboardType === 'electricity' ? '#eab308' : '#3b82f6';
                let backgroundColor = currentDashboardType === 'electricity' ? 'rgba(234, 179, 8, 0.2)' : 'rgba(59, 130, 246, 0.2)';

                // Check if the selected meter is an Ashghal meter and apply beige color
                if (currentDashboardType === 'water' && selectedMeterId !== 'summary' && 
                    (selectedMeterId === ashghalIrrigationTankId || selectedMeterId === ashghalROPlantId || selectedMeterId === ashghalMainMeterId || selectedMeterId === '15215002323')) {
                    borderColor = '#d4a762';
                    backgroundColor = 'rgba(212, 167, 98, 0.2)';
                }

                // Special handling for Irrigation Tank meter in monthly/yearly charts
                if (currentDashboardType === 'water' && selectedMeterId === ashghalIrrigationTankId && (viewModes[currentViewModeIndex] === 'monthly' || viewModes[currentViewModeIndex] === 'yearly')) {
                    unit = 'Dm³'; // Override unit to Dm³
                    label = `${selectedMeterId} (${getMeterConfig(selectedMeterId).location[currentLanguage]})`; // Use full location for label
                } else if (selectedMeterId !== 'summary') { // For other individual meters
                    label = `${selectedMeterId} (${getMeterConfig(selectedMeterId).location[currentLanguage]})`;
                }

                datasets.push({ label: `${label} (${unit})`, data: dataPointsSingleMeter, borderColor: borderColor, backgroundColor: backgroundColor, fill: true, tension: 0.3 });
            }

            return { labels, datasets };
        }

        function generateYearlyData(numYears) {
            console.log(`DEBUG: generateYearlyData for ${numYears} years.`);
            const selectedMeterId = meterChartFilter.value;
            const labels = []; 
            let dataPointsKahramaa = []; 
            let dataPointsAshghal = []; 
            let dataPointsSingleMeter = []; 
            let datasets = []; 

            const currentMetersConfig = allMetersData[currentDashboardType];
            const allDefinedMeters = getAllMetersFlatInCurrentConfig();
            const ashghalIrrigationTankId = 'K22223190003505907';
            const ashghalROPlantId = 'KF-18540133';
            const ashghalMainMeterId = '19ACI 005333';


            // Iterate from the selected year backwards by number of years
            for (let i = numYears - 1; i >= 0; i--) {
                const loopDate = new Date(selectedDate.getFullYear() - i, 0, 1); // First day of the year
                labels.push(loopDate.getFullYear().toString());

                if (selectedMeterId === 'summary') {
                    if (currentDashboardType === 'water') {
                        // Calculate aggregated yearly consumption for Kahramaa main meter
                        const { consumption: yearlyKahramaaTotal } = calculateAggregatedConsumptionForCategory('1219053', loopDate, 'yearly'); 
                        dataPointsKahramaa.push(parseFloat(yearlyKahramaaTotal.toFixed(2)));

                        // Calculate aggregated yearly consumption for Ashghal total: sum ONLY Irrigation and RO Plant.
                        let yearlyAshghalTotal = 0;
                        const year = loopDate.getFullYear();
                        const startDay = new Date(year, 0, 1);
                        const endDay = new Date(year, 11, 31);
                        let tempDate = new Date(startDay);
                        while (tempDate <= endDay) {
                            yearlyAshghalTotal += getDailyConsumptionForAnyMeter(ashghalIrrigationTankId, tempDate, true); // Sum in M3
                            yearlyAshghalTotal += getDailyConsumptionForAnyMeter(ashghalROPlantId, tempDate, true);   // Sum in M3
                            tempDate.setDate(tempDate.getDate() + 1);
                        }
                        dataPointsAshghal.push(parseFloat(yearlyAshghalTotal.toFixed(2)));
                    } else if (currentDashboardType === 'electricity') {
                        let yearlyTotalConsumption = 0;
                        allDefinedMeters.forEach(meterEntry => {
                            // Sum yearly consumption for each electricity meter
                            yearlyTotalConsumption += calculateAggregatedConsumptionForCategory(meterEntry.id, loopDate, 'yearly').consumption;
                        });
                        dataPointsSingleMeter.push(parseFloat(yearlyTotalConsumption.toFixed(2)));
                    }
                } else { 
                    // For individual meter, calculate its yearly consumption
                    const { consumption: yearlyTotalConsumption } = calculateAggregatedConsumptionForCategory(selectedMeterId, loopDate, 'yearly'); 
                    dataPointsSingleMeter.push(parseFloat(yearlyTotalConsumption.toFixed(2)));
                }
            }

            if (selectedMeterId === 'summary' && currentDashboardType === 'water') {
                datasets.push({
                    label: `${translations[currentLanguage].category_kahramaa} (${currentMetersConfig.summaryUnit})`,
                    data: dataPointsKahramaa,
                    borderColor: '#3b82f6', 
                    backgroundColor: 'rgba(59, 130, 246, 0.2)',
                    fill: true,
                    tension: 0.3
                });
                datasets.push({
                    label: `${translations[currentLanguage].category_ashghal} (${currentMetersConfig.summaryUnit})`,
                    data: dataPointsAshghal,
                    borderColor: '#d4a762', /* Light brown color */
                    backgroundColor: 'rgba(212, 167, 98, 0.2)',
                    fill: true,
                    tension: 0.3
                });
            } else {
                let label = selectedMeterId === 'summary' ? translations[currentLanguage].summaryTitle : selectedMeterId;
                let unit = currentMetersConfig.summaryUnit; // Default unit
                let borderColor = currentDashboardType === 'electricity' ? '#eab308' : '#3b82f6';
                let backgroundColor = currentDashboardType === 'electricity' ? 'rgba(234, 179, 8, 0.2)' : 'rgba(59, 130, 246, 0.2)';

                // Check if the selected meter is an Ashghal meter and apply beige color
                if (currentDashboardType === 'water' && selectedMeterId !== 'summary' && 
                    (selectedMeterId === ashghalIrrigationTankId || selectedMeterId === ashghalROPlantId || selectedMeterId === ashghalMainMeterId || selectedMeterId === '15215002323')) {
                    borderColor = '#d4a762';
                    backgroundColor = 'rgba(212, 167, 98, 0.2)';
                }

                // Special handling for Irrigation Tank meter in monthly/yearly charts
                if (currentDashboardType === 'water' && selectedMeterId === ashghalIrrigationTankId && (viewModes[currentViewModeIndex] === 'monthly' || viewModes[currentViewModeIndex] === 'yearly')) {
                    unit = 'Dm³'; // Override unit to Dm³
                    label = `${selectedMeterId} (${getMeterConfig(selectedMeterId).location[currentLanguage]})`; // Use full location for label
                } else if (selectedMeterId !== 'summary') { // For other individual meters
                    label = `${selectedMeterId} (${getMeterConfig(selectedMeterId).location[currentLanguage]})`;
                }

                datasets.push({ label: `${label} (${unit})`, data: dataPointsSingleMeter, borderColor: borderColor, backgroundColor: backgroundColor, fill: true, tension: 0.3 });
            }

            return { labels, datasets };
        }

        function updateViewModeLabel() {
            console.log(`DEBUG: Updating view mode label. Current index: ${currentViewModeIndex}`);
            const currentViewMode = viewModes[currentViewModeIndex];
            let labelText = '';
            switch(currentViewMode) {
                case 'daily_7': labelText = translations[currentLanguage].view_daily_7; break;
                case 'daily_30': labelText = translations[currentLanguage].view_daily_30; break;
                case 'monthly': labelText = translations[currentLanguage].view_month + 'View'; break; 
                case 'yearly': labelText = translations[currentLanguage].view_yearly + 'View'; break; 
            }
            viewModeLabel.textContent = labelText;
            console.log("DEBUG: View mode label updated.");
        }

        function updateChart() {
            console.log("DEBUG: updateChart function started.");
            if (consumptionChart) consumptionChart.destroy();

            const currentViewMode = viewModes[currentViewModeIndex];
            let chartData, chartType = 'bar';

            meterChartFilter.style.display = 'block'; 

            chartTitleEl.textContent = translations[currentLanguage].chartTitle;

            switch(currentViewMode) {
                case 'daily_30': 
                    chartData = generateDailyData(30); 
                    chartType = 'line'; 
                    break;
                case 'daily_7': 
                    chartData = generateDailyData(7); 
                    chartType = 'bar'; 
                    break;
                case 'monthly': 
                    chartData = generateMonthlyData(12);
                    chartType = 'bar';
                    break;
                case 'yearly': 
                    chartData = generateYearlyData(5);
                    chartType = 'bar';
                    break;
                default: 
                    chartData = { labels: [], datasets: [] }; 
            }

            // Determine the y-axis title dynamically
            let yAxisTitleText = '';
            if (meterChartFilter.value === 'summary' && currentDashboardType === 'water') {
                yAxisTitleText = `${translations[currentLanguage].consumption} (${allMetersData[currentDashboardType].summaryUnit})`;
            } else if (meterChartFilter.value !== 'summary') {
                const selectedMeterConfig = getMeterConfig(meterChartFilter.value);
                if (selectedMeterConfig && selectedMeterConfig.id === 'K22223190003505907' && (viewModes[currentViewModeIndex] === 'monthly' || viewModes[currentViewModeIndex] === 'yearly')) {
                    yAxisTitleText = `${translations[currentLanguage].consumption} (Dm³)`
                } else {
                    yAxisTitleText = `${translations[currentLanguage].consumption} (${allMetersData[currentDashboardType].summaryUnit})`;
                }
            } else {
                yAxisTitleText = `${translations[currentLanguage].consumption} (${allMetersData[currentDashboardType].summaryUnit})`;
            }


            consumptionChart = new Chart(document.getElementById('consumptionChart').getContext('2d'), {
                type: chartType, 
                data: chartData,
                options: { 
                    responsive: true, 
                    maintainAspectRatio: false, 
                    scales: { 
                        y: { 
                            beginAtZero: true, 
                            ticks: { color: getComputedStyle(document.body).getPropertyValue('color') }, /* Tick colors to match body text color */
                            grid: { color: document.body.classList.contains('dark-mode') ? '#4a5568' : '#e2e8f0' }, /* Grid colors to match light/dark background */
                            title: {
                                display: true,
                                text: yAxisTitleText, // Use the dynamically determined y-axis title
                                color: getComputedStyle(document.body).getPropertyValue('color') /* Title colors to match body text color */
                            }
                        }, 
                        x: { ticks: { color: getComputedStyle(document.body).getPropertyValue('color') }, grid: { color: document.body.classList.contains('dark-mode') ? '#4a5568' : '#e2e8f0' }} /* Tick and grid colors to match light/dark background */
                    }, 
                    plugins: { 
                        legend: { 
                            labels: { 
                                color: getComputedStyle(document.body).getPropertyValue('color'), /* Legend label colors to match body text color */
                                font: { family: 'Inter' } // Always Inter
                            } 
                        } 
                    } 
                }
            });
            console.log("DEBUG: updateChart function finished.");
        }

        // Helper function to get the formatted period label for the PDF report
        function getPeriodLabelForReport() {
            const currentViewMode = viewModes[currentViewModeIndex];
            let periodLabel = '';
            const options = {
                day: '2-digit',
                month: 'long',
                year: 'numeric'
            };

            // Determine locale based on current language for date formatting in PDF
            const locale = currentLanguage === 'ar' ? 'ar-EG' : 'en-GB'; // Changed to 'en-GB' for Day Month Year format

            switch (currentViewMode) {
                case 'daily_7':
                case 'daily_30':
                    periodLabel = `${translations[currentLanguage].pdfDateLabel}: ${new Intl.DateTimeFormat(locale, options).format(selectedDate)}`;
                    break;
                case 'monthly':
                    periodLabel = `${translations[currentLanguage].pdfMonthLabel}: ${new Intl.DateTimeFormat(locale, { month: 'long', year: 'numeric' }).format(selectedDate)}`;
                    break;
                case 'yearly':
                    periodLabel = `${translations[currentLanguage].pdfYearLabel}: ${selectedDate.getFullYear()}`;
                    break;
            }
            return periodLabel;
        }

        async function generatePdfReport() {
            console.log("DEBUG: generatePdfReport function started.");
            printBtn.disabled = true;
            printBtn.innerHTML = `<i class="bi bi-hourglass-split animate-spin"></i><span id="print-btn-text">${translations[currentLanguage].generating_report}</span>`;

            // Hide non-printable elements
            document.querySelectorAll('.no-print').forEach(el => el.style.display = 'none');

            const { jsPDF } = window.jspdf;
            const doc = new jsPDF('p', 'mm', 'a4');
            const margin = 15;
            let y = margin;
            const lineHeight = 7;

            // Set default font for the document (can be 'helvetica', 'times', 'courier')
            doc.setFont("helvetica", "normal"); 

            // Main Title
            doc.setFontSize(22);
            doc.text(allMetersData[currentDashboardType].dashboardTitle[currentLanguage], margin, y);
            y += lineHeight * 2;

            // Period Label
            doc.setFontSize(12); // Smaller font for period label
            doc.text(getPeriodLabelForReport(), margin, y);
            y += lineHeight * 2; // Add more space after the period label

            doc.setFontSize(14);

            if (currentDashboardType === 'water') {
                let totalKahramaaConsumption = 0;
                let totalAshghalConsumption = 0;

                const { consumption: kahramaaMainComplexCons } = calculateAggregatedConsumptionForCategory('1219053', selectedDate, viewModes[currentViewModeIndex]);
                totalKahramaaConsumption = kahramaaMainComplexCons;

                // Recalculate totalAshghalConsumption here to explicitly only include Irrigation and RO Plant
                const ashghalIrrigationTankId = 'K22223190003505907';
                const ashghalROPlantId = 'KF-18540133';

                let tempAshghalConsumption = 0;
                const year = selectedDate.getFullYear();
                const month = selectedDate.getMonth();
                let startDay, endDay;

                if (viewModes[currentViewModeIndex] === 'daily_7' || viewModes[currentViewModeIndex] === 'daily_30') {
                    tempAshghalConsumption += getDailyConsumptionForAnyMeter(ashghalIrrigationTankId, selectedDate, true); // Sum in M3
                    tempAshghalConsumption += getDailyConsumptionForAnyMeter(ashghalROPlantId, selectedDate, true); // Sum in M3
                } else if (viewModes[currentViewModeIndex] === 'monthly') {
                    startDay = new Date(year, month, 1);
                    endDay = new Date(year, month + 1, 0);
                    let tempDate = new Date(startDay);
                    while (tempDate <= endDay) {
                        tempAshghalConsumption += getDailyConsumptionForAnyMeter(ashghalIrrigationTankId, tempDate, true); // Sum in M3
                        tempAshghalConsumption += getDailyConsumptionForAnyMeter(ashghalROPlantId, tempDate, true); // Sum in M3
                        tempDate.setDate(tempDate.getDate() + 1);
                    }
                } else if (viewModes[currentViewModeIndex] === 'yearly') {
                    startDay = new Date(year, 0, 1);
                    endDay = new Date(year, 11, 31);
                    let tempDate = new Date(startDay);
                    while (tempDate <= endDay) {
                        tempAshghalConsumption += getDailyConsumptionForAnyMeter(ashghalIrrigationTankId, tempDate, true); // Sum in M3
                        tempAshghalConsumption += getDailyConsumptionForAnyMeter(ashghalROPlantId, tempDate, true); // Sum in M3
                        tempDate.setDate(tempDate.getDate() + 1);
                    }
                }
                totalAshghalConsumption = tempAshghalConsumption;


                doc.text(`${translations[currentLanguage].category_kahramaa}:`, margin, y);
                y += lineHeight;
                doc.text(`${translations[currentLanguage].summaryTitle}: ${totalKahramaaConsumption.toFixed(2)} ${allMetersData[currentDashboardType].summaryUnit}`, margin, y);
                y += lineHeight * 2; // Add extra space after Kahramaa section

                doc.text(`${translations[currentLanguage].category_ashghal}:`, margin, y);
                y += lineHeight;
                doc.text(`${translations[currentLanguage].summaryTitle}: ${totalAshghalConsumption.toFixed(2)} ${allMetersData[currentDashboardType].summaryUnit}`, margin, y);
                y += lineHeight * 2; // Add extra space after Ashghal section

                doc.text(`${translations[currentLanguage].summaryTitle} (${translations[currentLanguage].toggle_to_water}): ${(totalKahramaaConsumption + totalAshghalConsumption).toFixed(2)} ${allMetersData[currentDashboardType].summaryUnit}`, margin, y);
                y += lineHeight * 2;
            } else {
                doc.text(`${translations[currentLanguage].summaryTitle}: ${summaryTotalEl.textContent}`, margin, y);
                y += lineHeight * 2;
            }


            doc.setFontSize(12);
            const currentMetersConfig = allMetersData[currentDashboardType];
            const isDailyView = ['daily_7', 'daily_30'].includes(viewModes[currentViewModeIndex]);
            const headerStyle = { fillColor: [226, 232, 240], textColor: [74, 85, 104], fontStyle: 'bold' }; /* Header colors to match light background */
            // Adjust header style for dark mode if active
            if (document.body.classList.contains('dark-mode')) {
                headerStyle.fillColor = [45, 55, 72]; // Darker background for header
                headerStyle.textColor = [226, 232, 240]; // Lighter text for header
            }


            const getAllMetersFlat = (config) => {
                let meters = [];
                for (const categoryKey in config.categories) {
                    const category = config.categories[categoryKey];
                    for (const subCategoryKey in category.sub_categories) {
                        meters = meters.concat(category.sub_categories[subCategoryKey].panels);
                    }
                }
                return meters;
            };
            const allMetersFlat = getAllMetersFlat(currentMetersConfig);

            const ashghalIrrigationTankId = 'K22223190003505907'; // Define for use in PDF report logic

            for (const meter of allMetersFlat) {
                // Ensure WF is displayed correctly as "Water Features" in PDF
                const meterLocationText = meter.location[currentLanguage];

                if (y + 50 > doc.internal.pageSize.height - margin) {
                    doc.addPage();
                    y = margin;
                }

                doc.text(`${meter.id} (${meterLocationText})`, margin, y);
                y += lineHeight;

                let head = [];
                let body = [];
                let unitForReport = currentMetersConfig.summaryUnit;

                if (isDailyView) {
                    head = [[translations[currentLanguage].prevReading, translations[currentLanguage].currentReading, translations[currentLanguage].consumption]];

                    let currentReadingValue, prevReadingValue, consumptionValue;

                    if (meter.id === 'WF') {
                         currentReadingValue = getMeterDataForDate('WF', selectedDate).value;
                         prevReadingValue = getMeterDataForDate('WF', getPreviousDate(selectedDate)).value;

                         if (currentReadingValue !== undefined && prevReadingValue !== undefined) {
                             consumptionValue = (currentReadingValue - prevReadingValue).toFixed(2); // Corrected this line
                         } else {
                             consumptionValue = "N/A";
                         }
                    } else {
                        currentReadingValue = getMeterDataForDate(meter.id, selectedDate)?.value || 'N/A';
                        prevReadingValue = getMeterDataForDate(meter.id, getPreviousDate(selectedDate))?.value || 'N/A';

                        // Use getDailyConsumptionForAnyMeter for consistency in PDF
                        consumptionValue = getDailyConsumptionForAnyMeter(meter.id, selectedDate, true).toFixed(2);
                    }
                    body.push([prevReadingValue, currentReadingValue, `${consumptionValue} ${unitForReport}`]);
                } else {
                    // For monthly/yearly view in PDF, use aggregated consumption
                    head = [[translations[currentLanguage].consumption]];
                    const { consumption: aggConsumption } = calculateAggregatedConsumptionForCategory(meter.id, selectedDate, viewModes[currentViewModeIndex]); 

                    // Determine unit for PDF report
                    if (meter.id === ashghalIrrigationTankId) {
                        unitForReport = 'Dm³'; // Irrigation tank shows Dm³ in monthly/yearly reports
                    } else {
                        unitForReport = currentMetersConfig.summaryUnit; // Other meters use M³ or KWH
                    }

                    body.push([`${aggConsumption.toLocaleString(undefined, {maximumFractionDigits: 0})} ${unitForReport}`]);
                }

                doc.autoTable({
                    startY: y,
                    head: head,
                    body: body,
                    theme: 'grid', 
                    headStyles: headerStyle,
                    styles: {
                        font: "helvetica", 
                        textColor: document.body.classList.contains('dark-mode') ? [226, 232, 240] : [74, 85, 104], /* Text color to match light/dark background */
                        fillColor: document.body.classList.contains('dark-mode') ? [45, 55, 72] : [248, 250, 252] /* Background color to match light/dark background */
                    },
                    margin: { left: margin, right: margin },
                    didDrawPage: function(data) {
                        y = data.cursor.y;
                    }
                });
                y = doc.autoTable.previous.finalY + margin;
            }

            if (consumptionChart) {
                if (y + (doc.internal.pageSize.width - margin * 2) * 0.75 > doc.internal.pageSize.height - margin) { 
                    doc.addPage();
                    y = margin;
                }

                doc.setFontSize(14);
                doc.text(translations[currentLanguage].chartTitle, margin, y);
                y += lineHeight;

                const chartCanvas = document.getElementById('consumptionChart');
                // Ensure chart is fully rendered before trying to get its image
                if (!chartCanvas || chartCanvas.width === 0 || chartCanvas.height === 0) {
                    console.warn("Chart canvas is not available or has zero dimensions. Skipping chart image in PDF.");
                } else {
                    const imgData = chartCanvas.toDataURL('image/png', 1.0); // Use 1.0 for maximum quality
                    const imgWidth = doc.internal.pageSize.width - (margin * 2); 
                    // Calculate imgHeight based on aspect ratio to avoid stretching
                    const imgHeight = chartCanvas.height * imgWidth / chartCanvas.width;

                    doc.addImage(imgData, 'PNG', margin, y, imgWidth, imgHeight);
                }
            }

            doc.save(`${currentDashboardType}_Report_${formatDateToYYYYMMDD(new Date())}.pdf`);

            // Restore non-printable elements display
            document.querySelectorAll('.no-print').forEach(el => el.style.display = '');
            printBtn.disabled = false;
            printBtn.innerHTML = `<i class="bi bi-printer-fill"></i><span id="print-btn-text">${translations[currentLanguage].printBtn}</span>`;
            console.log("DEBUG: generatePdfReport function finished.");
        }


        // Event Listeners
        prevBtn.addEventListener('click', () => { 
            console.log("DEBUG: prevBtn clicked.");
            const currentViewMode = viewModes[currentViewModeIndex];
            switch (currentViewMode) {
                case 'daily_7':
                case 'daily_30':
                    selectedDate.setDate(selectedDate.getDate() - 1); 
                    break;
                case 'monthly':
                    selectedDate.setMonth(selectedDate.getMonth() - 1);
                    break;
                case 'yearly':
                    selectedDate.setFullYear(selectedDate.getFullYear() - 1);
                    break;
            }
            renderAll(); 
        });
        nextBtn.addEventListener('click', () => { 
            console.log("DEBUG: nextBtn clicked.");
            const currentViewMode = viewModes[currentViewModeIndex];
            switch (currentViewMode) {
                case 'daily_7':
                case 'daily_30':
                    selectedDate.setDate(selectedDate.getDate() + 1); 
                    break;
                case 'monthly':
                    selectedDate.setMonth(selectedDate.getMonth() + 1);
                    break;

                case 'yearly':
                    selectedDate.setFullYear(selectedDate.getFullYear() + 1);
                    break;
            }
            renderAll(); 
        });

        prevViewBtn.addEventListener('click', () => { 
            console.log("DEBUG: prevViewBtn clicked.");
            currentViewModeIndex = (currentViewModeIndex - 1 + viewModes.length) % viewModes.length; 
            selectedDate = new Date();
            meterChartFilter.value = 'summary';
            renderAll(); 
        });
        nextViewBtn.addEventListener('click', () => { 
            console.log("DEBUG: nextViewModeBtn clicked.");
            currentViewModeIndex = (currentViewModeIndex + 1) % viewModes.length; 
            selectedDate = new Date();
            meterChartFilter.value = 'summary';
            renderAll(); 
        });

        dashboardToggleBtn.addEventListener('click', () => {
            console.log("DEBUG: dashboardToggleBtn clicked.");
            currentDashboardType = (currentDashboardType === 'water') ? 'electricity' : 'water';
            selectedDate = new Date();
            meterChartFilter.value = 'summary'; // Reset filter to summary
            currentViewModeIndex = 0;
            populateMeterDropdown();
            setupFirestoreListener();
            renderAll();
        });

        meterChartFilter.addEventListener('change', updateChart);
        printBtn.addEventListener('click', generatePdfReport);

        // Dark Mode Toggle Event Listener
        darkModeToggleBtn.addEventListener('click', () => {
            document.body.classList.toggle('dark-mode');
            applyDarkModePreference(); // Re-apply preference to update icon
            updateChart(); // Re-render chart to apply new colors
        });


        function setupFirestoreListener() {
            if (unsubscribeFirestore) {
                unsubscribeFirestore();
                console.log("DEBUG: Unsubscribed from previous Firestore listener.");
            }

            const collectionPath = allMetersData[currentDashboardType].collection;
            const readingsCollectionRef = window.firestoreCollection(db, collectionPath);

            console.log(`DEBUG: Setting up Firestore listener for collection: ${collectionPath}`);
            unsubscribeFirestore = window.firestoreOnSnapshot(readingsCollectionRef, (snapshot) => {
                const liveReadings = {};
                if (snapshot.empty) {
                    console.warn(`DEBUG: Firestore snapshot for ${collectionPath} is empty. No documents found.`);
                }
                snapshot.forEach(doc => { 
                    const dateId = doc.id;
                    const data = doc.data();
                    liveReadings[dateId] = data; 
                });
                allReadings = liveReadings;
                console.log("DEBUG: Fetched readings from Firestore:", allReadings);
                if (loaderContainer) loaderContainer.remove();
                renderAll();
            }, (error) => {
                console.error("FIRESTORE ERROR:", error);
                if (loaderContainer) {
                    loaderContainer.innerHTML = `<p class="text-red-500">${translations[currentLanguage].dbError}<br>Error: ${error.message}</p>`;
                }
            });
        }

        // Initial setup
        setupFirestoreListener();
        setLanguage('en'); // Set language to English initially
    });
    </script>
</body>
</html> 