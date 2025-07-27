import React, { useState, useEffect, useRef } from 'react';
import { CheckCircle, Circle, AlertCircle, FileText, Download, Upload, Save, Printer, FileDown, Clock, Calendar, Thermometer, Cloud, Edit3 } from 'lucide-react';

export default function HVACCommissioningComplete() {
  const [showCover, setShowCover] = useState(true);
  const [activeTab, setActiveTab] = useState('projectInfo');
  const [lastSaved, setLastSaved] = useState(null);
  const fileInputRef = useRef(null);

  // State management
  const [projectInfo, setProjectInfo] = useState({
    storeNumber: '11172',
    storeAddress: '',
    cityStateZip: '',
    testDate: new Date().toISOString().split('T')[0],
    startTime: '',
    endTime: '',
    outdoorTemp: '',
    indoorTemp: '',
    humidity: '',
    weather: 'Clear'
  });

  const [assetInfo, setAssetInfo] = useState({
    'split-1': { make: '', model: '', serial: '', assetNum: '', serviceArea: '' },
    'split-2': { make: '', model: '', serial: '', assetNum: '', serviceArea: '' },
    'split-3': { make: '', model: '', serial: '', assetNum: '', serviceArea: '' },
    'split-4': { make: '', model: '', serial: '', assetNum: '', serviceArea: '' }
  });

  const [checkedItems, setCheckedItems] = useState({});
  const [functionalTests, setFunctionalTests] = useState({});
  const [testResults, setTestResults] = useState({});
  const [notes, setNotes] = useState({});
  const [fieldNotes, setFieldNotes] = useState('');
  const [acceptanceCriteria, setAcceptanceCriteria] = useState({});
  const [signatures, setSignatures] = useState({
    contractor: { name: '', title: '', date: '', signed: false },
    commissioning: { name: '', title: '', date: '', signed: false },
    walgreens: { name: '', title: '', date: '', signed: false, acceptance: '' }
  });

  // Checklist data
  const prefunctionalChecklists = {
    airHandling: {
      title: "A. Air-Handling Units (40 RLA)",
      items: [
        "Clearance for service (≥ 30 in sides, 36 in front) and heater service gap 30 in",
        "Unit orientation (horizontal/vertical) matches drawings",
        "Fan wheel spins freely; set-screws torqued",
        "Filters 2 in MERV-8 installed, gaskets intact",
        "Coil piping pitched to drain; unions & isolation valves installed",
        "Electric heater wiring landed on factory lugs; branch circuit sized to MCA/MOCP",
        "SAV fan speed jumpers set per external static: 1.99 in wg (6 ton) / 1.41 in wg (12.5 ton)",
        "Differential pressure tubes for filter status installed"
      ]
    },
    economizer: {
      title: "B. Economizer Sections",
      items: [
        "Outdoor & return dampers fully close (< 4 CFM/ft² leak) when de-energized",
        "Siemens POL224 controller wired; enthalpy sensor and CO₂ sensor mounted, calibration date tags"
      ]
    },
    condensing: {
      title: "C. Condensing Units (38 AXZ)",
      items: [
        "Suction/liquid lines insulated & pressure-tested; nitrogen holding charge purged; leak checked",
        "Service valves back-seated; proper oil level; crankcase heater powered 24 h prior to start",
        "Line lengths ≤ 30 ft as designed; verify traps/kickers where vertical drops occur"
      ]
    },
    steam: {
      title: "D. Steam Distribution Coils",
      items: [
        "Dirt pocket with blow-down valve at coil inlet; vacuum breaker at outlet",
        "Control valve fails closed on loss of power",
        "Freeze-stat bulb strapped on leaving air side and interlocked to fan starter"
      ]
    }
  };

  const functionalTestData = [
    { no: 1, prerequisite: "Pre-checks complete, power ON", action: "Command fan \"AUTO – Occupied\"", expected: "Fan runs Low; airflow ≈ 1,800 CFM; amperage ≤ 6.7 A", record: "RPM, ESP, Amps" },
    { no: 2, prerequisite: "Fan running", action: "Simulate filter loading to 1.0 in wg", expected: "Fan alarm after 15 s; OA damper closes to min.", record: "ΔP value, alarm time" },
    { no: 3, prerequisite: "OAT = 90°F, RAT = 75°F", action: "Disable economizer, raise space stat 3°F", expected: "Stage-1 comp ON; T_sa drops below 55°F in 5 min", record: "Suction/Disch ψ, T_sa, kW" },
    { no: 4, prerequisite: "Continue Test 3", action: "Raise stat another 2°F (load)", expected: "Stage-2 comp ON; cond. fans second speed", record: "Same" },
    { no: 5, prerequisite: "OAT = 55°F, RAT = 75°F", action: "Enable economizer, disable cooling", expected: "OA damper modulates; compressors remain OFF", record: "Damper % open, T_mix" },
    { no: 6, prerequisite: "OAT = 40°F", action: "Occupancy heat call", expected: "Electric heat St-1 ON; T_sa rises ≈ 76°F", record: "kW draw (~9.2 kW)" },
    { no: 7, prerequisite: "Extend heat call 2 min", action: "Electric heat St-2 ON; supply temp ≤ 90°F", expected: "kW ~18.3 kW (Split-4)", record: "" },
    { no: 8, prerequisite: "AC-1 only, OAT = 30°F", action: "Disable electric heat, enable steam valve", expected: "Valve modulates; T_sa 90 ± 2°F; condensate 6–8 gpm", record: "ΔT coil, steam ψ" },
    { no: 9, prerequisite: "Trip HP switch", action: "Compressor stops; Comfort-Alert logs code; auto-reset after manual reset", expected: "Fault code ID", record: "" },
    { no: 10, prerequisite: "Press POL224 Test", action: "Economizer cycles to 100% OA then closes; alarm clears", expected: "Damper pos., LED status", record: "" },
    { no: 11, prerequisite: "Simulate refrigerant leak", action: "Unit shuts compressors & fans; alarm relay closes", expected: "Time to shutdown, LED code", record: "" },
    { no: 12, prerequisite: "Power loss 1 min", action: "On return, fan restart delay 2 min; compressors anti-short-cycle 5 min", expected: "Event log", record: "" }
  ];

  const acceptanceCriteriaList = [
    "Cooling capacity within ±5% of 66.3 MBH (6 ton) or 135 MBH (12.5 ton) performance tables",
    "Electric heater kW within ±10% of nameplate (10 / 20 kW)",
    "Steam coil leaving air temp reaches ≥ 90°F within 3 min and stable ±2°F",
    "Economizer provides at least 25% OA at minimum position and maintains T_mix ≤ 55°F without compressors when OAT ≤ 55°F",
    "All safety shutdowns and alarms annunciated at BAS and local unit controller"
  ];

  // Auto-save functionality
  useEffect(() => {
    const interval = setInterval(() => {
      saveToLocalStorage();
    }, 30000); // Auto-save every 30 seconds

    return () => clearInterval(interval);
  }, [checkedItems, functionalTests, testResults, notes, fieldNotes, projectInfo, assetInfo, acceptanceCriteria, signatures]);

  const saveToLocalStorage = () => {
    const data = {
      projectInfo,
      assetInfo,
      checkedItems,
      functionalTests,
      testResults,
      notes,
      fieldNotes,
      acceptanceCriteria,
      signatures,
      savedAt: new Date().toISOString()
    };
    localStorage.setItem('hvacCommissioningData', JSON.stringify(data));
    setLastSaved(new Date());
  };

  const loadFromLocalStorage = () => {
    const saved = localStorage.getItem('hvacCommissioningData');
    if (saved) {
      const data = JSON.parse(saved);
      setProjectInfo(data.projectInfo || projectInfo);
      setAssetInfo(data.assetInfo || assetInfo);
      setCheckedItems(data.checkedItems || {});
      setFunctionalTests(data.functionalTests || {});
      setTestResults(data.testResults || {});
      setNotes(data.notes || {});
      setFieldNotes(data.fieldNotes || '');
      setAcceptanceCriteria(data.acceptanceCriteria || {});
      setSignatures(data.signatures || signatures);
      setLastSaved(data.savedAt ? new Date(data.savedAt) : null);
    }
  };

  useEffect(() => {
    loadFromLocalStorage();
  }, []);

  const handleCheck = (section, item, unit) => {
    const key = `${section}-${item}-${unit}`;
    setCheckedItems(prev => ({
      ...prev,
      [key]: prev[key] === 'checked' ? 'nc' : prev[key] === 'nc' ? 'na' : prev[key] === 'na' ? '' : 'checked'
    }));
  };

  const handleTestResult = (testNo, unit, value) => {
    const key = `${testNo}-${unit}`;
    setTestResults(prev => ({
      ...prev,
      [key]: value
    }));
  };

  const handleNote = (key, value) => {
    setNotes(prev => ({
      ...prev,
      [key]: value
    }));
  };

  const getCompletionPercentage = (unit) => {
    let totalItems = 0;
    let completedItems = 0;

    // Count prefunctional items
    Object.entries(prefunctionalChecklists).forEach(([sectionKey, section]) => {
      section.items.forEach((_, index) => {
        totalItems++;
        const key = `${sectionKey}-${index}-${unit}`;
        if (checkedItems[key]) completedItems++;
      });
    });

    // Count functional tests
    functionalTestData.forEach((test) => {
      totalItems++;
      const key = `${test.no}-${unit}`;
      if (testResults[key] && testResults[key].trim()) completedItems++;
    });

    return totalItems > 0 ? Math.round((completedItems / totalItems) * 100) : 0;
  };

  const exportToJSON = () => {
    const data = {
      metadata: {
        reportType: 'HVAC Commissioning Test Script',
        storeNumber: projectInfo.storeNumber,
        testDate: projectInfo.testDate,
        generatedAt: new Date().toISOString(),
        generatedBy: 'Leigh Scott Solutions'
      },
      projectInfo,
      assetInfo,
      checkedItems,
      functionalTests,
      testResults,
      notes,
      fieldNotes,
      acceptanceCriteria,
      signatures,
      summary: {
        'split-1': { progress: getCompletionPercentage('split-1') + '%' },
        'split-2': { progress: getCompletionPercentage('split-2') + '%' },
        'split-3': { progress: getCompletionPercentage('split-3') + '%' },
        'split-4': { progress: getCompletionPercentage('split-4') + '%' }
      }
    };

    const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `commissioning-${projectInfo.storeNumber}-${new Date().toISOString().split('T')[0]}.json`;
    a.click();
    URL.revokeObjectURL(url);
  };

  const importFromJSON = (event) => {
    const file = event.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (e) => {
      try {
        const data = JSON.parse(e.target.result);
        if (data.projectInfo) setProjectInfo(data.projectInfo);
        if (data.assetInfo) setAssetInfo(data.assetInfo);
        if (data.checkedItems) setCheckedItems(data.checkedItems);
        if (data.functionalTests) setFunctionalTests(data.functionalTests);
        if (data.testResults) setTestResults(data.testResults);
        if (data.notes) setNotes(data.notes);
        if (data.fieldNotes !== undefined) setFieldNotes(data.fieldNotes);
        if (data.acceptanceCriteria) setAcceptanceCriteria(data.acceptanceCriteria);
        if (data.signatures) setSignatures(data.signatures);
        alert('Data imported successfully!');
      } catch (error) {
        alert('Error importing file. Please ensure it\'s a valid JSON export.');
      }
    };
    reader.readAsText(file);
  };

  const exportToWord = () => {
    // Create HTML content for Word export
    const htmlContent = `
      <!DOCTYPE html>
      <html>
      <head>
        <meta charset="utf-8">
        <title>HVAC Commissioning Report - Store #${projectInfo.storeNumber}</title>
        <style>
          body { font-family: Arial, sans-serif; margin: 40px; }
          h1, h2, h3 { color: #1e3a8a; }
          table { border-collapse: collapse; width: 100%; margin: 20px 0; }
          th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
          th { background-color: #f2f2f2; }
          .header { text-align: center; margin-bottom: 40px; }
          .section { margin: 30px 0; page-break-inside: avoid; }
          .signature-line { border-bottom: 1px solid black; width: 300px; margin-top: 40px; }
          @media print { .section { page-break-inside: avoid; } }
        </style>
      </head>
      <body>
        <div class="header">
          <h1>Leigh Scott Solutions</h1>
          <h2>HVAC Commissioning Test Report</h2>
          <h3>Walgreens Store #${projectInfo.storeNumber}</h3>
          <p>Test Date: ${projectInfo.testDate}</p>
        </div>

        <div class="section">
          <h2>Project Information</h2>
          <table>
            <tr><td><strong>Store Address:</strong></td><td>${projectInfo.storeAddress}</td></tr>
            <tr><td><strong>City, State, ZIP:</strong></td><td>${projectInfo.cityStateZip}</td></tr>
            <tr><td><strong>Test Date:</strong></td><td>${projectInfo.testDate}</td></tr>
            <tr><td><strong>Start Time:</strong></td><td>${projectInfo.startTime}</td></tr>
            <tr><td><strong>End Time:</strong></td><td>${projectInfo.endTime}</td></tr>
            <tr><td><strong>Outdoor Temp:</strong></td><td>${projectInfo.outdoorTemp}°F</td></tr>
            <tr><td><strong>Indoor Temp:</strong></td><td>${projectInfo.indoorTemp}°F</td></tr>
            <tr><td><strong>Humidity:</strong></td><td>${projectInfo.humidity}%</td></tr>
            <tr><td><strong>Weather:</strong></td><td>${projectInfo.weather}</td></tr>
          </table>
        </div>

        <div class="section">
          <h2>Asset Information</h2>
          <table>
            <tr>
              <th>Unit</th><th>Make</th><th>Model</th><th>Serial</th><th>Asset #</th><th>Service Area</th>
            </tr>
            ${Object.entries(assetInfo).map(([unit, info]) => `
              <tr>
                <td>${unit.toUpperCase()}</td>
                <td>${info.make}</td>
                <td>${info.model}</td>
                <td>${info.serial}</td>
                <td>${info.assetNum}</td>
                <td>${info.serviceArea}</td>
              </tr>
            `).join('')}
          </table>
        </div>

        <div class="section">
          <h2>Pre-Functional Checklist Results</h2>
          ${Object.entries(prefunctionalChecklists).map(([sectionKey, section]) => `
            <h3>${section.title}</h3>
            <table>
              <tr>
                <th>Item</th>
                <th>Split-1</th>
                <th>Split-2</th>
                <th>Split-3</th>
                <th>Split-4</th>
              </tr>
              ${section.items.map((item, index) => `
                <tr>
                  <td>${item}</td>
                  ${['split-1', 'split-2', 'split-3', 'split-4'].map(unit => {
                    const status = checkedItems[`${sectionKey}-${index}-${unit}`];
                    return `<td>${status === 'checked' ? '✓' : status === 'nc' ? '✗' : status === 'na' ? 'N/A' : '-'}</td>`;
                  }).join('')}
                </tr>
              `).join('')}
            </table>
          `).join('')}
        </div>

        <div class="section">
          <h2>Functional Test Results</h2>
          <table>
            <tr>
              <th>Test</th>
              <th>Description</th>
              <th>Split-1</th>
              <th>Split-2</th>
              <th>Split-3</th>
              <th>Split-4</th>
            </tr>
            ${functionalTestData.map(test => `
              <tr>
                <td>Test ${test.no}</td>
                <td>${test.action}</td>
                ${['split-1', 'split-2', 'split-3', 'split-4'].map(unit => 
                  `<td>${testResults[`${test.no}-${unit}`] || '-'}</td>`
                ).join('')}
              </tr>
            `).join('')}
          </table>
        </div>

        <div class="section">
          <h2>Field Notes</h2>
          <p>${fieldNotes || 'No field notes recorded.'}</p>
        </div>

        <div class="section">
          <h2>Acceptance Criteria</h2>
          <ul>
            ${acceptanceCriteriaList.map((criterion, index) => 
              `<li>${acceptanceCriteria[index] ? '✓' : '☐'} ${criterion}</li>`
            ).join('')}
          </ul>
        </div>

        <div class="section">
          <h2>Signatures</h2>
          <div style="margin-top: 40px;">
            <h3>Installing Contractor</h3>
            <p>Name: ${signatures.contractor.name}</p>
            <p>Title: ${signatures.contractor.title}</p>
            <p>Date: ${signatures.contractor.date}</p>
            <div class="signature-line"></div>
          </div>
          <div style="margin-top: 40px;">
            <h3>Commissioning Agent</h3>
            <p>Name: ${signatures.commissioning.name}</p>
            <p>Title: ${signatures.commissioning.title}</p>
            <p>Date: ${signatures.commissioning.date}</p>
            <div class="signature-line"></div>
          </div>
          <div style="margin-top: 40px;">
            <h3>Walgreens Representative</h3>
            <p>Name: ${signatures.walgreens.name}</p>
            <p>Title: ${signatures.walgreens.title}</p>
            <p>Date: ${signatures.walgreens.date}</p>
            <p>Acceptance Status: ${signatures.walgreens.acceptance}</p>
            <div class="signature-line"></div>
          </div>
        </div>
      </body>
      </html>
    `;

    // Create blob and download
    const blob = new Blob([htmlContent], { type: 'application/msword' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `HVAC-Commissioning-${projectInfo.storeNumber}-${new Date().toISOString().split('T')[0]}.doc`;
    a.click();
    URL.revokeObjectURL(url);
  };

  const renderCheckbox = (section, index, unit) => {
    const key = `${section}-${index}-${unit}`;
    const status = checkedItems[key];
    
    return (
      <button
        onClick={() => handleCheck(section, index, unit)}
        className={`w-8 h-8 rounded-full flex items-center justify-center transition-all transform hover:scale-110 ${
          status === 'checked' ? 'bg-green-500 text-white shadow-lg' : 
          status === 'nc' ? 'bg-red-500 text-white shadow-lg' : 
          status === 'na' ? 'bg-gray-500 text-white shadow-lg' :
          'bg-gray-200 hover:bg-gray-300 text-gray-500'
        }`}
        title={status === 'checked' ? 'Pass' : status === 'nc' ? 'Fail' : status === 'na' ? 'N/A' : 'Click to check'}
      >
        {status === 'checked' && <CheckCircle size={18} />}
        {status === 'nc' && <AlertCircle size={18} />}
        {status === 'na' && <span className="text-xs font-bold">N/A</span>}
        {!status && <Circle size={18} />}
      </button>
    );
  };

  if (showCover) {
    return (
      <div className="min-h-screen bg-gradient-to-br from-blue-900 to-blue-700 flex items-center justify-center p-8">
        <div className="text-center text-white max-w-4xl mx-auto">
          <div className="mb-8 inline-block p-8 border-4 border-white border-dashed rounded-lg bg-white/10 backdrop-blur">
            <FileText size={80} className="mx-auto" />
          </div>
          <h1 className="text-5xl font-bold mb-4">Leigh Scott Solutions</h1>
          <h2 className="text-3xl mb-8">HVAC Commissioning Test Script</h2>
          <div className="text-xl mb-4">
            DX Split Systems 38 AXZ / 40 RLA<br />
            with Electric Heat & Steam Coils
          </div>
          <div className="text-2xl font-semibold mb-8">
            Walgreens Store #{projectInfo.storeNumber}
          </div>
          <div className="text-lg mb-8">
            {new Date().toLocaleDateString('en-US', { 
              weekday: 'long', 
              year: 'numeric', 
              month: 'long', 
              day: 'numeric' 
            })}
          </div>
          <button
            onClick={() => setShowCover(false)}
            className="bg-white text-blue-900 px-8 py-4 rounded-lg font-semibold text-lg hover:bg-blue-50 transition-colors shadow-xl"
          >
            Begin Commissioning
          </button>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-50">
      {/* Header */}
      <div className="bg-white shadow-lg sticky top-0 z-50">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex justify-between items-center py-4">
            <div>
              <h1 className="text-2xl font-bold text-gray-900">
                Walgreens #{projectInfo.storeNumber} - Commissioning
              </h1>
              <p className="text-sm text-gray-600">DX Split Systems with Electric Heat & Steam Coils</p>
            </div>
            <div className="flex items-center gap-3">
              {lastSaved && (
                <span className="text-xs text-gray-500 flex items-center gap-1">
                  <Clock size={12} />
                  Last saved: {lastSaved.toLocaleTimeString()}
                </span>
              )}
              <input
                ref={fileInputRef}
                type="file"
                accept=".json"
                onChange={importFromJSON}
                className="hidden"
              />
              <button
                onClick={() => fileInputRef.current?.click()}
                className="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors flex items-center gap-2 text-sm"
              >
                <Upload size={16} />
                Import
              </button>
              <button
                onClick={saveToLocalStorage}
                className="bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700 transition-colors flex items-center gap-2 text-sm"
              >
                <Save size={16} />
                Save
              </button>
              <button
                onClick={exportToJSON}
                className="bg-purple-600 text-white px-4 py-2 rounded-lg hover:bg-purple-700 transition-colors flex items-center gap-2 text-sm"
              >
                <Download size={16} />
                Export JSON
              </button>
              <button
                onClick={exportToWord}
                className="bg-indigo-600 text-white px-4 py-2 rounded-lg hover:bg-indigo-700 transition-colors flex items-center gap-2 text-sm"
              >
                <FileDown size={16} />
                Export to Word
              </button>
              <button
                onClick={() => window.print()}
                className="bg-gray-600 text-white px-4 py-2 rounded-lg hover:bg-gray-700 transition-colors flex items-center gap-2 text-sm"
              >
                <Printer size={16} />
                Print
              </button>
            </div>
          </div>
        </div>
      </div>

      {/* Navigation Tabs */}
      <div className="bg-white border-b">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <nav className="flex gap-8">
            {[
              { id: 'projectInfo', label: 'Project Info', icon: FileText },
              { id: 'assets', label: 'Assets', icon: Edit3 },
              { id: 'prefunctional', label: 'Pre-Functional', icon: CheckCircle },
              { id: 'functional', label: 'Functional Tests', icon: AlertCircle },
              { id: 'acceptance', label: 'Acceptance', icon: CheckCircle }
            ].map(tab => (
              <button
                key={tab.id}
                onClick={() => setActiveTab(tab.id)}
                className={`py-4 px-2 border-b-2 transition-colors flex items-center gap-2 ${
                  activeTab === tab.id 
                    ? 'border-blue-600 text-blue-600' 
                    : 'border-transparent text-gray-500 hover:text-gray-700'
                }`}
              >
                <tab.icon size={20} />
                {tab.label}
              </button>
            ))}
          </nav>
        </div>
      </div>

      {/* Content */}
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        {/* Progress Cards */}
        <div className="grid grid-cols-1 md:grid-cols-4 gap-4 mb-8">
          {['split-1', 'split-2', 'split-3', 'split-4'].map(unit => (
            <div key={unit} className="bg-white rounded-lg shadow p-6">
              <h3 className="text-lg font-semibold text-gray-800 mb-1">
                {unit.toUpperCase()}
              </h3>
              <p className="text-sm text-gray-600 mb-4">
                {unit === 'split-4' ? '12.5 ton / 20 kW' : '6 ton / 10 kW'}
              </p>
              <div className="w-full bg-gray-200 rounded-full h-2 mb-2">
                <div 
                  className="bg-blue-600 h-2 rounded-full transition-all duration-500"
                  style={{ width: `${getCompletionPercentage(unit)}%` }}
                />
              </div>
              <p className="text-sm text-gray-600">
                {getCompletionPercentage(unit)}% Complete
              </p>
            </div>
          ))}
        </div>

        {/* Project Information Tab */}
        {activeTab === 'projectInfo' && (
          <div className="bg-white rounded-lg shadow-lg p-6">
            <h2 className="text-2xl font-bold text-gray-800 mb-6">Project Information</h2>
            
            <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div>
                <h3 className="text-lg font-semibold text-gray-700 mb-4 flex items-center gap-2">
                  <FileText size={20} />
                  Site Details
                </h3>
                <div className="space-y-4">
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">Store Number</label>
                    <input
                      type="text"
                      value={projectInfo.storeNumber}
                      onChange={(e) => setProjectInfo({...projectInfo, storeNumber: e.target.value})}
                      className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">Store Address</label>
                    <input
                      type="text"
                      value={projectInfo.storeAddress}
                      onChange={(e) => setProjectInfo({...projectInfo, storeAddress: e.target.value})}
                      className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                      placeholder="Enter store address"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">City, State, ZIP</label>
                    <input
                      type="text"
                      value={projectInfo.cityStateZip}
                      onChange={(e) => setProjectInfo({...projectInfo, cityStateZip: e.target.value})}
                      className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                      placeholder="City, State ZIP"
                    />
                  </div>
                </div>
              </div>

              <div>
                <h3 className="text-lg font-semibold text-gray-700 mb-4 flex items-center gap-2">
                  <Calendar size={20} />
                  Test Information
                </h3>
                <div className="space-y-4">
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">Test Date</label>
                    <input
                      type="date"
                      value={projectInfo.testDate}
                      onChange={(e) => setProjectInfo({...projectInfo, testDate: e.target.value})}
                      className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                    />
                  </div>
                  <div className="grid grid-cols-2 gap-4">
                    <div>
                      <label className="block text-sm font-medium text-gray-700 mb-1">Start Time</label>
                      <input
                        type="time"
                        value={projectInfo.startTime}
                        onChange={(e) => setProjectInfo({...projectInfo, startTime: e.target.value})}
                        className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                      />
                    </div>
                    <div>
                      <label className="block text-sm font-medium text-gray-700 mb-1">End Time</label>
                      <input
                        type="time"
                        value={projectInfo.endTime}
                        onChange={(e) => setProjectInfo({...projectInfo, endTime: e.target.value})}
                        className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                      />
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <div className="mt-6 pt-6 border-t">
              <h3 className="text-lg font-semibold text-gray-700 mb-4 flex items-center gap-2">
                <Thermometer size={20} />
                Test Conditions
              </h3>
              <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">Outdoor Temp (°F)</label>
                  <input
                    type="number"
                    value={projectInfo.outdoorTemp}
                    onChange={(e) => setProjectInfo({...projectInfo, outdoorTemp: e.target.value})}
                    className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                    placeholder="°F"
                  />
                </div>
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">Indoor Temp (°F)</label>
                  <input
                    type="number"
                    value={projectInfo.indoorTemp}
                    onChange={(e) => setProjectInfo({...projectInfo, indoorTemp: e.target.value})}
                    className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                    placeholder="°F"
                  />
                </div>
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">Humidity (%)</label>
                  <input
                    type="number"
                    value={projectInfo.humidity}
                    onChange={(e) => setProjectInfo({...projectInfo, humidity: e.target.value})}
                    className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                    placeholder="%"
                  />
                </div>
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1 flex items-center gap-1">
                    <Cloud size={16} />
                    Weather
                  </label>
                  <select
                    value={projectInfo.weather}
                    onChange={(e) => setProjectInfo({...projectInfo, weather: e.target.value})}
                    className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                  >
                    <option value="Clear">Clear</option>
                    <option value="Cloudy">Cloudy</option>
                    <option value="Rain">Rain</option>
                    <option value="Snow">Snow</option>
                  </select>
                </div>
              </div>
            </div>
          </div>
        )}

        {/* Asset Information Tab */}
        {activeTab === 'assets' && (
          <div className="bg-white rounded-lg shadow-lg p-6">
            <h2 className="text-2xl font-bold text-gray-800 mb-6">Asset Information</h2>
            
            <div className="overflow-x-auto">
              <table className="w-full">
                <thead>
                  <tr className="bg-gray-50">
                    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Field</th>
                    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Split-1</th>
                    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Split-2</th>
                    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Split-3</th>
                    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Split-4</th>
                  </tr>
                </thead>
                <tbody className="bg-white divide-y divide-gray-200">
                  {['make', 'model', 'serial', 'assetNum', 'serviceArea'].map(field => (
                    <tr key={field}>
                      <td className="px-4 py-3 font-medium text-gray-900 capitalize">
                        {field === 'assetNum' ? 'Asset #' : field === 'serviceArea' ? 'Service Area' : field}
                      </td>
                      {['split-1', 'split-2', 'split-3', 'split-4'].map(unit => (
                        <td key={unit} className="px-4 py-3">
                          <input
                            type="text"
                            value={assetInfo[unit][field]}
                            onChange={(e) => setAssetInfo({
                              ...assetInfo,
                              [unit]: {...assetInfo[unit], [field]: e.target.value}
                            })}
                            className="w-full px-2 py-1 border rounded focus:ring-2 focus:ring-blue-500"
                            placeholder={`Enter ${field}`}
                          />
                        </td>
                      ))}
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>
        )}

        {/* Pre-Functional Tab */}
        {activeTab === 'prefunctional' && (
          <div className="space-y-8">
            {Object.entries(prefunctionalChecklists).map(([key, section]) => (
              <div key={key} className="bg-white rounded-lg shadow-lg">
                <div className="bg-gray-50 px-6 py-4 rounded-t-lg">
                  <h3 className="text-xl font-semibold text-gray-800">{section.title}</h3>
                </div>
                <div className="p-6">
                  <div className="overflow-x-auto">
                    <table className="w-full">
                      <thead>
                        <tr className="border-b">
                          <th className="pb-2 text-left text-sm font-medium text-gray-700">Inspection Item</th>
                          <th className="pb-2 text-center text-sm font-medium text-gray-700">Split-1</th>
                          <th className="pb-2 text-center text-sm font-medium text-gray-700">Split-2</th>
                          <th className="pb-2 text-center text-sm font-medium text-gray-700">Split-3</th>
                          <th className="pb-2 text-center text-sm font-medium text-gray-700">Split-4</th>
                        </tr>
                      </thead>
                      <tbody>
                        {section.items.map((item, index) => (
                          <tr key={index} className="border-b">
                            <td className="py-3 pr-4 text-sm">{item}</td>
                            {['split-1', 'split-2', 'split-3', 'split-4'].map(unit => (
                              <td key={unit} className="py-3 text-center">
                                {renderCheckbox(key, index, unit)}
                                {checkedItems[`${key}-${index}-${unit}`] === 'nc' && (
                                  <input
                                    type="text"
                                    placeholder="Issue..."
                                    className="mt-2 w-full px-2 py-1 text-xs border rounded"
                                    value={notes[`${key}-${index}-${unit}`] || ''}
                                    onChange={(e) => handleNote(`${key}-${index}-${unit}`, e.target.value)}
                                  />
                                )}
                              </td>
                            ))}
                          </tr>
                        ))}
                      </tbody>
                    </table>
                  </div>
                </div>
              </div>
            ))}
          </div>
        )}

        {/* Functional Tests Tab */}
        {activeTab === 'functional' && (
          <div className="bg-white rounded-lg shadow-lg p-6">
            <h2 className="text-2xl font-bold text-gray-800 mb-6">Functional Test Results</h2>
            
            <div className="overflow-x-auto">
              <table className="w-full">
                <thead>
                  <tr className="bg-gray-50">
                    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Test</th>
                    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Action</th>
                    <th className="px-4 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Split-1</th>
                    <th className="px-4 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Split-2</th>
                    <th className="px-4 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Split-3</th>
                    <th className="px-4 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Split-4</th>
                  </tr>
                </thead>
                <tbody className="bg-white divide-y divide-gray-200">
                  {functionalTestData.map((test) => (
                    <tr key={test.no}>
                      <td className="px-4 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                        Test {test.no}
                      </td>
                      <td className="px-4 py-4 text-sm text-gray-700">
                        <div className="font-medium">{test.action}</div>
                        <div className="text-xs text-gray-500 mt-1">Expected: {test.expected}</div>
                        {test.record && (
                          <div className="text-xs text-blue-600 mt-1">Record: {test.record}</div>
                        )}
                      </td>
                      {['split-1', 'split-2', 'split-3', 'split-4'].map(unit => (
                        <td key={unit} className="px-4 py-4 text-center">
                          <textarea
                            className="w-full px-2 py-1 border rounded text-sm resize-none"
                            rows="3"
                            placeholder="Enter results..."
                            value={testResults[`${test.no}-${unit}`] || ''}
                            onChange={(e) => handleTestResult(test.no, unit, e.target.value)}
                          />
                        </td>
                      ))}
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>

            <div className="mt-8 p-6 bg-yellow-50 rounded-lg border border-yellow-200">
              <h3 className="text-lg font-semibold text-yellow-900 mb-3">Field Notes & Issues</h3>
              <textarea
                className="w-full px-4 py-3 border border-yellow-300 rounded-lg resize-none focus:ring-2 focus:ring-yellow-500"
                rows="5"
                placeholder="Record any issues, deviations, or special observations during testing..."
                value={fieldNotes}
                onChange={(e) => setFieldNotes(e.target.value)}
              />
            </div>
          </div>
        )}

        {/* Acceptance Tab */}
        {activeTab === 'acceptance' && (
          <div className="space-y-8">
            <div className="bg-white rounded-lg shadow-lg p-6">
              <h2 className="text-2xl font-bold text-gray-800 mb-6">Acceptance Criteria</h2>
              
              <div className="space-y-3">
                {acceptanceCriteriaList.map((criterion, index) => (
                  <label key={index} className="flex items-start gap-3 p-3 hover:bg-gray-50 rounded-lg transition-colors">
                    <input
                      type="checkbox"
                      checked={acceptanceCriteria[index] || false}
                      onChange={(e) => setAcceptanceCriteria({
                        ...acceptanceCriteria,
                        [index]: e.target.checked
                      })}
                      className="mt-1 w-5 h-5 text-blue-600 rounded focus:ring-blue-500"
                    />
                    <span className="text-gray-700">{criterion}</span>
                  </label>
                ))}
              </div>
            </div>

            <div className="bg-white rounded-lg shadow-lg p-6">
              <h2 className="text-2xl font-bold text-gray-800 mb-6">Certification & Signatures</h2>
              
              <p className="text-gray-600 mb-8">
                By signing below, all parties certify that the equipment has been installed, tested, and commissioned 
                in accordance with the manufacturer's specifications, applicable codes, and the requirements outlined 
                in this commissioning test script.
              </p>

              <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div className="border rounded-lg p-6">
                  <h3 className="text-lg font-semibold text-gray-800 mb-4">Installing Contractor</h3>
                  <div className="space-y-3">
                    <div>
                      <label className="block text-sm font-medium text-gray-700 mb-1">Name & Title</label>
                      <input
                        type="text"
                        value={signatures.contractor.name}
                        onChange={(e) => setSignatures({
                          ...signatures,
                          contractor: {...signatures.contractor, name: e.target.value}
                        })}
                        className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                        placeholder="John Doe, Senior Technician"
                      />
                    </div>
                    <div>
                      <label className="block text-sm font-medium text-gray-700 mb-1">Date</label>
                      <input
                        type="date"
                        value={signatures.contractor.date}
                        onChange={(e) => setSignatures({
                          ...signatures,
                          contractor: {...signatures.contractor, date: e.target.value}
                        })}
                        className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                      />
                    </div>
                    <div>
                      <label className="flex items-center gap-2">
                        <input
                          type="checkbox"
                          checked={signatures.contractor.signed}
                          onChange={(e) => setSignatures({
                            ...signatures,
                            contractor: {...signatures.contractor, signed: e.target.checked}
                          })}
                          className="w-4 h-4 text-blue-600 rounded focus:ring-blue-500"
                        />
                        <span className="text-sm text-gray-700">Signed</span>
                      </label>
                    </div>
                  </div>
                </div>

                <div className="border rounded-lg p-6">
                  <h3 className="text-lg font-semibold text-gray-800 mb-4">Commissioning Agent</h3>
                  <div className="space-y-3">
                    <div>
                      <label className="block text-sm font-medium text-gray-700 mb-1">Name & Title</label>
                      <input
                        type="text"
                        value={signatures.commissioning.name}
                        onChange={(e) => setSignatures({
                          ...signatures,
                          commissioning: {...signatures.commissioning, name: e.target.value}
                        })}
                        className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                        placeholder="Jane Smith, CxA"
                      />
                    </div>
                    <div>
                      <label className="block text-sm font-medium text-gray-700 mb-1">Date</label>
                      <input
                        type="date"
                        value={signatures.commissioning.date}
                        onChange={(e) => setSignatures({
                          ...signatures,
                          commissioning: {...signatures.commissioning, date: e.target.value}
                        })}
                        className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                      />
                    </div>
                    <div>
                      <label className="flex items-center gap-2">
                        <input
                          type="checkbox"
                          checked={signatures.commissioning.signed}
                          onChange={(e) => setSignatures({
                            ...signatures,
                            commissioning: {...signatures.commissioning, signed: e.target.checked}
                          })}
                          className="w-4 h-4 text-blue-600 rounded focus:ring-blue-500"
                        />
                        <span className="text-sm text-gray-700">Signed</span>
                      </label>
                    </div>
                  </div>
                </div>
              </div>

              <div className="border rounded-lg p-6 mt-6">
                <h3 className="text-lg font-semibold text-gray-800 mb-4">Walgreens Representative</h3>
                <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">Name & Title</label>
                    <input
                      type="text"
                      value={signatures.walgreens.name}
                      onChange={(e) => setSignatures({
                        ...signatures,
                        walgreens: {...signatures.walgreens, name: e.target.value}
                      })}
                      className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                      placeholder="Bob Johnson, Facilities Manager"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">Date</label>
                    <input
                      type="date"
                      value={signatures.walgreens.date}
                      onChange={(e) => setSignatures({
                        ...signatures,
                        walgreens: {...signatures.walgreens, date: e.target.value}
                      })}
                      className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                    />
                  </div>
                </div>
                <div className="mt-4">
                  <label className="block text-sm font-medium text-gray-700 mb-2">Acceptance Status</label>
                  <div className="flex gap-6">
                    {['Accepted', 'Conditional', 'Rejected'].map(status => (
                      <label key={status} className="flex items-center gap-2">
                        <input
                          type="radio"
                          name="acceptance"
                          value={status}
                          checked={signatures.walgreens.acceptance === status}
                          onChange={(e) => setSignatures({
                            ...signatures,
                            walgreens: {...signatures.walgreens, acceptance: e.target.value}
                          })}
                          className="w-4 h-4 text-blue-600 focus:ring-blue-500"
                        />
                        <span className="text-sm text-gray-700">{status}</span>
                      </label>
                    ))}
                  </div>
                </div>
                <div className="mt-4">
                  <label className="flex items-center gap-2">
                    <input
                      type="checkbox"
                      checked={signatures.walgreens.signed}
                      onChange={(e) => setSignatures({
                        ...signatures,
                        walgreens: {...signatures.walgreens, signed: e.target.checked}
                      })}
                      className="w-4 h-4 text-blue-600 rounded focus:ring-blue-500"
                    />
                    <span className="text-sm text-gray-700">Signed</span>
                  </label>
                </div>
              </div>
            </div>
          </div>
        )}
      </div>

      {/* Print Styles */}
      <style jsx>{`
        @media print {
          .no-print {
            display: none !important;
          }
          body {
            print-color-adjust: exact;
            -webkit-print-color-adjust: exact;
          }
          .page-break {
            page-break-after: always;
          }
        }
      `}</style>
    </div>
  );
}
