import React, { useState, useEffect, useRef } from 'react';
import { Calendar, Clock, Printer, Download, CheckCircle, XCircle, MinusCircle, AlertCircle, Save } from 'lucide-react';

const CommissioningTestScript = () => {
  // State for project information
  const [projectInfo, setProjectInfo] = useState({
    storeNumber: '11172',
    storeAddress: '',
    cityStateZip: '',
    testDate: '',
    startTime: '',
    endTime: '',
    outdoorTemp: '',
    indoorTemp: '',
    humidity: '',
    weather: 'Clear'
  });

  // State for contractor information
  const [contractorInfo, setContractorInfo] = useState({
    installingContractor: '',
    leadTechnician: '',
    licenseNumber: '',
    phoneNumber: '',
    email: '',
    projectManager: '',
    commissioningAgent: '',
    commissioningCompany: '',
    certificateNumber: ''
  });

  // State for equipment asset information
  const [assetInfo, setAssetInfo] = useState({
    'split-1': { make: '', model: '', serial: '', assetNum: '', serviceArea: '' },
    'split-2': { make: '', model: '', serial: '', assetNum: '', serviceArea: '' },
    'split-3': { make: '', model: '', serial: '', assetNum: '', serviceArea: '' },
    'split-4': { make: '', model: '', serial: '', assetNum: '', serviceArea: '' }
  });

  // State for checklists
  const [checklists, setChecklists] = useState({});
  
  // State for test results
  const [testResults, setTestResults] = useState({});
  
  // State for notes
  const [fieldNotes, setFieldNotes] = useState('');
  
  // State for acceptance
  const [acceptance, setAcceptance] = useState({
    status: '',
    signatures: {
      contractor: { signature: '', name: '', date: '' },
      commissioning: { signature: '', name: '', date: '' },
      walgreens: { signature: '', name: '', date: '', status: '' }
    }
  });

  // Auto-save functionality
  const [lastSaved, setLastSaved] = useState(null);
  
  // Calculate progress for each unit
  const calculateProgress = (unit) => {
    const unitChecks = Object.entries(checklists)
      .filter(([key]) => key.includes(unit))
      .map(([, value]) => value);
    
    if (unitChecks.length === 0) return 0;
    
    const completed = unitChecks.filter(v => v !== null).length;
    return (completed / unitChecks.length) * 100;
  };

  // Handle checklist updates
  const updateChecklist = (section, unit, value) => {
    const key = `${section}-${unit}`;
    setChecklists(prev => ({
      ...prev,
      [key]: value
    }));
  };

  // Handle test result updates
  const updateTestResult = (test, unit, value) => {
    const key = `${test}-${unit}`;
    setTestResults(prev => ({
      ...prev,
      [key]: value
    }));
  };

  // Print functionality
  const handlePrint = () => {
    // Add print-specific class to body
    document.body.classList.add('printing');
    
    // Trigger print
    window.print();
    
    // Remove print class after a delay
    setTimeout(() => {
      document.body.classList.remove('printing');
    }, 500);
  };

  // Export functionality - generates JSON download
  const generateReport = () => {
    const reportData = {
      metadata: {
        reportType: 'HVAC Commissioning Test Script',
        storeNumber: projectInfo.storeNumber,
        testDate: projectInfo.testDate,
        generatedAt: new Date().toISOString()
      },
      projectInfo,
      contractorInfo,
      assetInfo,
      checklists,
      testResults,
      fieldNotes,
      acceptance,
      summary: {
        'split-1': { progress: calculateProgress('split-1') + '%' },
        'split-2': { progress: calculateProgress('split-2') + '%' },
        'split-3': { progress: calculateProgress('split-3') + '%' },
        'split-4': { progress: calculateProgress('split-4') + '%' }
      }
    };
    
    // Create a blob with the JSON data
    const dataStr = JSON.stringify(reportData, null, 2);
    const blob = new Blob([dataStr], { type: 'application/json' });
    
    // Create download link
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.download = `commissioning-report-${projectInfo.storeNumber}-${new Date().toISOString().split('T')[0]}.json`;
    
    // Trigger download
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    
    // Clean up
    URL.revokeObjectURL(url);
  };

  // Helper function to convert data to CSV rows
  const convertToCSVRows = (data) => {
    if (data.length === 0) return [];
    
    const headers = Object.keys(data[0]);
    const rows = [headers.join(',')];
    
    data.forEach(item => {
      const values = headers.map(header => {
        const value = item[header] || '';
        // Escape quotes and wrap in quotes if contains comma
        return value.toString().includes(',') ? `"${value.toString().replace(/"/g, '""')}"` : value;
      });
      rows.push(values.join(','));
    });
    
    return rows;
  };

  // Export to CSV
  const exportToCSV = () => {
    // Prepare checklist data for CSV
    const checklistData = [];
    const units = ['split-1', 'split-2', 'split-3', 'split-4'];
    
    // Air handling checklist
    airHandlingChecklist.forEach(item => {
      const row = { 'Inspection Item': item.label, 'Category': 'Air Handling' };
      units.forEach(unit => {
        row[unit.toUpperCase()] = checklists[`airhandling-${item.id}-${unit}`] || 'Not Checked';
      });
      checklistData.push(row);
    });
    
    // Prepare test results for CSV
    const testResultsData = [];
    functionalTests.forEach(test => {
      const row = { 'Test': test.name, 'Description': test.desc };
      units.forEach(unit => {
        row[unit.toUpperCase()] = testResults[`${test.id}-${unit}`] || 'No Data';
      });
      testResultsData.push(row);
    });
    
    // Convert to CSV format
    const csvContent = [
      'COMMISSIONING TEST REPORT',
      `Store #${projectInfo.storeNumber}`,
      `Test Date: ${projectInfo.testDate}`,
      '',
      'CHECKLIST RESULTS',
      ...convertToCSVRows(checklistData),
      '',
      'TEST RESULTS',
      ...convertToCSVRows(testResultsData),
      '',
      'FIELD NOTES',
      fieldNotes || 'No notes recorded'
    ].join('\n');
    
    // Create and download CSV
    const blob = new Blob([csvContent], { type: 'text/csv' });
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.download = `commissioning-checklist-${projectInfo.storeNumber}-${new Date().toISOString().split('T')[0]}.csv`;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    URL.revokeObjectURL(url);
  };

  // Save data to memory (in a real app, this would be localStorage or a backend)
  const saveData = () => {
    const allData = {
      projectInfo,
      contractorInfo,
      assetInfo,
      checklists,
      testResults,
      fieldNotes,
      acceptance,
      savedAt: new Date().toISOString()
    };
    
    // In a real app, you'd save to localStorage or backend
    // For demo, we'll just update the last saved timestamp
    setLastSaved(new Date().toLocaleString());
    
    // Simulated save notification
    const notification = document.createElement('div');
    notification.className = 'fixed bottom-4 right-4 bg-green-600 text-white px-4 py-2 rounded-lg shadow-lg z-50';
    notification.textContent = 'Progress saved!';
    document.body.appendChild(notification);
    
    setTimeout(() => {
      notification.remove();
    }, 2000);
  };

  // Import data from JSON file
  const importData = (event) => {
    const file = event.target.files[0];
    if (!file) return;
    
    const reader = new FileReader();
    reader.onload = (e) => {
      try {
        const data = JSON.parse(e.target.result);
        
        // Restore all data
        if (data.projectInfo) setProjectInfo(data.projectInfo);
        if (data.contractorInfo) setContractorInfo(data.contractorInfo);
        if (data.assetInfo) setAssetInfo(data.assetInfo);
        if (data.checklists) setChecklists(data.checklists);
        if (data.testResults) setTestResults(data.testResults);
        if (data.fieldNotes) setFieldNotes(data.fieldNotes);
        if (data.acceptance) setAcceptance(data.acceptance);
        
        // Show success notification
        const notification = document.createElement('div');
        notification.className = 'fixed bottom-4 right-4 bg-blue-600 text-white px-4 py-2 rounded-lg shadow-lg z-50';
        notification.textContent = 'Data imported successfully!';
        document.body.appendChild(notification);
        
        setTimeout(() => {
          notification.remove();
        }, 2000);
      } catch (error) {
        alert('Error importing file. Please ensure it\'s a valid JSON export from this tool.');
      }
    };
    reader.readAsText(file);
  };

  // Auto-save every 30 seconds when there are changes
  useEffect(() => {
    const interval = setInterval(() => {
      if (Object.keys(checklists).length > 0 || Object.keys(testResults).length > 0) {
        saveData();
      }
    }, 30000);
    
    return () => clearInterval(interval);
  }, [checklists, testResults]);

  // Checklist items
  const airHandlingChecklist = [
    { id: 'clearance', label: 'Clearance for service (≥30" sides, 36" front)' },
    { id: 'orientation', label: 'Unit orientation matches drawings' },
    { id: 'fanWheel', label: 'Fan wheel spins freely; set-screws torqued' },
    { id: 'filters', label: 'Filters 2" MERV-8 installed, gaskets intact' },
    { id: 'coilPiping', label: 'Coil piping pitched to drain; unions & valves installed' },
    { id: 'electricWiring', label: 'Electric heater wiring on factory lugs; MCA/MOCP sized' },
    { id: 'savFan', label: 'SAV fan jumpers set: 1.99"wg (6T) / 1.41"wg (12.5T)' },
    { id: 'diffPressure', label: 'Differential pressure tubes for filter status installed' }
  ];

  const economizerChecklist = [
    { id: 'damperClose', label: 'OA & RA dampers fully close (<4 CFM/ft² leak)' },
    { id: 'siemensWired', label: 'Siemens POL224 wired; sensors mounted & calibrated' }
  ];

  const condensingChecklist = [
    { id: 'linesInsulated', label: 'Suction/liquid lines insulated & pressure-tested; nitrogen purged' },
    { id: 'serviceValves', label: 'Service valves back-seated; proper oil level' },
    { id: 'crankcaseHeater', label: 'Crankcase heater powered 24h prior to start' },
    { id: 'lineLengths', label: 'Line lengths ≤30 ft; traps/kickers at vertical drops' }
  ];

  const functionalTests = [
    { id: 'test1', name: 'Fan Operation', desc: 'AUTO-Occupied, verify Low speed', target: 'Target: 1,800 CFM / ≤6.7A' },
    { id: 'test2', name: 'Filter Loading', desc: 'Simulate 1.0" wg, alarm in 15s' },
    { id: 'test3', name: 'Cooling Stage 1', desc: 'OAT=90°F, RAT=75°F, +3°F stat', target: 'Target: T_sa <55°F in 5min' },
    { id: 'test4', name: 'Cooling Stage 2', desc: 'Additional +2°F load' },
    { id: 'test5', name: 'Economizer', desc: 'OAT=55°F, verify damper modulation' },
    { id: 'test6', name: 'Electric Heat St-1', desc: 'OAT=40°F, heat call', target: 'Target: ~9.2kW (6T) / ~18.3kW (12.5T)' },
    { id: 'test7', name: 'Electric Heat St-2', desc: 'Extended heat call 2min', target: 'Target: T_sa ≤90°F' },
    { id: 'test8', name: 'Safeties', desc: 'HP switch, refrigerant leak, POL224 test' },
    { id: 'test9', name: 'HP Switch Trip', desc: 'Trip high pressure switch', target: 'Verify: Comfort-Alert logs, auto-reset' },
    { id: 'test10', name: 'POL224 Test Mode', desc: 'Press POL224 Test button', target: 'Damper cycles 100% OA then closes' },
    { id: 'test11', name: 'Refrigerant Leak Test', desc: 'Simulate leak (sensor test button)', target: 'Unit shuts compressors & fans' },
    { id: 'test12', name: 'Power Loss Recovery', desc: 'Power loss 1 min', target: 'Fan delay 2min, compressor 5min' }
  ];

  // Render checkbox button
  const CheckButton = ({ value, onChange, options = ['yes', 'no', 'na'] }) => {
    const icons = {
      yes: <CheckCircle className="w-3 h-3" />,
      no: <XCircle className="w-3 h-3" />,
      na: <MinusCircle className="w-3 h-3" />
    };

    const colors = {
      yes: 'bg-green-500 hover:bg-green-600',
      no: 'bg-red-500 hover:bg-red-600',
      na: 'bg-gray-500 hover:bg-gray-600'
    };

    return (
      <div className="flex gap-2 justify-center">
        {options.map(option => (
          <button
            key={option}
            onClick={() => onChange(value === option ? null : option)}
            className={`w-8 h-8 rounded-full border-2 flex items-center justify-center transition-all ${
              value === option 
                ? `${colors[option]} text-white border-transparent` 
                : 'border-gray-300 hover:border-gray-400 text-gray-400'
            }`}
            title={option.toUpperCase()}
          >
            {value === option && icons[option]}
          </button>
        ))}
      </div>
    );
  };

  return (
    <>
      <style dangerouslySetInnerHTML={{ __html: `
        @media print {
          /* Hide non-printable elements */
          .printing button {
            display: none !important;
          }
          
          /* Reset backgrounds for print */
          .printing * {
            background-color: white !important;
            color: black !important;
          }
          
          /* Ensure tables fit on page */
          .printing table {
            font-size: 10pt !important;
            page-break-inside: avoid;
          }
          
          /* Headers should be visible on each page */
          .printing thead {
            display: table-header-group;
          }
          
          /* Avoid page breaks inside rows */
          .printing tr {
            page-break-inside: avoid;
          }
          
          /* Make borders more visible */
          .printing table, .printing th, .printing td {
            border: 1px solid #000 !important;
          }
          
          /* Hide shadows and unnecessary decorations */
          .printing .shadow-md,
          .printing .shadow-lg,
          .printing .shadow-sm {
            box-shadow: none !important;
          }
          
          /* Ensure inputs are visible */
          .printing input,
          .printing textarea,
          .printing select {
            border: 1px solid #666 !important;
            padding: 2px !important;
          }
          
          /* Make checkbox states visible */
          .printing .bg-green-500 {
            background-color: #10b981 !important;
            print-color-adjust: exact;
            -webkit-print-color-adjust: exact;
          }
          
          .printing .bg-red-500 {
            background-color: #ef4444 !important;
            print-color-adjust: exact;
            -webkit-print-color-adjust: exact;
          }
          
          .printing .bg-gray-500 {
            background-color: #6b7280 !important;
            print-color-adjust: exact;
            -webkit-print-color-adjust: exact;
          }
          
          /* Adjust layout for print */
          .printing .max-w-7xl {
            max-width: 100% !important;
          }
          
          /* Page breaks */
          .printing .page-break-before {
            page-break-before: always;
          }
          
          /* Hide sticky header on print */
          .printing .sticky {
            position: static !important;
          }
        }
      `}} />
      <div className="min-h-screen bg-gray-50">
        {/* Header */}
        <div className="bg-white border-b border-gray-200 sticky top-0 z-10 shadow-sm">
          <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div className="flex items-center justify-between h-16">
              <div>
                <h1 className="text-2xl font-bold text-gray-900">
                  Walgreens #{projectInfo.storeNumber} - Commissioning Test Script
                </h1>
                <p className="text-sm text-gray-600">
                  DX Split Systems 38 AXZ / 40 RLA with Electric Heat & Steam Coils
                </p>
              </div>
              <div className="flex items-center gap-4">
                {lastSaved && (
                  <span className="text-xs text-gray-500">
                    Last saved: {lastSaved}
                  </span>
                )}
                <div className="flex gap-2">
                  <input
                    type="file"
                    id="import-file"
                    accept=".json"
                    onChange={importData}
                    className="hidden"
                  />
                  <label
                    htmlFor="import-file"
                    className="flex items-center gap-2 px-4 py-2 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700 transition-colors cursor-pointer"
                  >
                    <Download className="w-4 h-4 rotate-180" />
                    Import
                  </label>
                  <button
                    onClick={saveData}
                    className="flex items-center gap-2 px-4 py-2 bg-gray-600 text-white rounded-lg hover:bg-gray-700 transition-colors"
                  >
                    <Save className="w-4 h-4" />
                    Save
                  </button>
                  <button
                    onClick={handlePrint}
                    className="flex items-center gap-2 px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
                  >
                    <Printer className="w-4 h-4" />
                    Print
                  </button>
                  <button
                    onClick={generateReport}
                    className="flex items-center gap-2 px-4 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700 transition-colors"
                  >
                    <Download className="w-4 h-4" />
                    Export JSON
                  </button>
                  <button
                    onClick={exportToCSV}
                    className="flex items-center gap-2 px-4 py-2 bg-purple-600 text-white rounded-lg hover:bg-purple-700 transition-colors"
                  >
                    <Download className="w-4 h-4" />
                    Export CSV
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>

        {/* Main Content */}
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
          {/* Project Information */}
          <div className="bg-white rounded-lg shadow-md mb-8">
            <div className="bg-blue-900 text-white px-6 py-4 rounded-t-lg">
              <h2 className="text-xl font-semibold">Project Information</h2>
            </div>
            <div className="p-6">
              <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                {/* Site Details */}
                <div>
                  <h3 className="text-lg font-semibold text-gray-700 mb-4">Site Details</h3>
                  <div className="space-y-4">
                    <div>
                      <label className="block text-sm font-medium text-gray-700 mb-1">Store Number</label>
                      <input
                        type="text"
                        value={projectInfo.storeNumber}
                        onChange={(e) => setProjectInfo({...projectInfo, storeNumber: e.target.value})}
                        className="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
                      />
                    </div>
                    <div>
                      <label className="block text-sm font-medium text-gray-700 mb-1">Store Address</label>
                      <input
                        type="text"
                        value={projectInfo.storeAddress}
                        onChange={(e) => setProjectInfo({...projectInfo, storeAddress: e.target.value})}
                        className="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
                        placeholder="Enter store address"
                      />
                    </div>
                    <div>
                      <label className="block text-sm font-medium text-gray-700 mb-1">City, State, ZIP</label>
                      <input
                        type="text"
                        value={projectInfo.cityStateZip}
                        onChange={(e) => setProjectInfo({...projectInfo, cityStateZip: e.target.value})}
                        className="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
                        placeholder="City, State ZIP"
                      />
                    </div>
                  </div>
                </div>

                {/* Test Information */}
                <div>
                  <h3 className="text-lg font-semibold text-gray-700 mb-4">Test Information</h3>
                  <div className="space-y-4">
                    <div>
                      <label className="block text-sm font-medium text-gray-700 mb-1">Test Date</label>
                      <input
                        type="date"
                        value={projectInfo.testDate}
                        onChange={(e) => setProjectInfo({...projectInfo, testDate: e.target.value})}
                        className="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
                      />
                    </div>
                    <div className="grid grid-cols-2 gap-4">
                      <div>
                        <label className="block text-sm font-medium text-gray-700 mb-1">Start Time</label>
                        <input
                          type="time"
                          value={projectInfo.startTime}
                          onChange={(e) => setProjectInfo({...projectInfo, startTime: e.target.value})}
                          className="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
                        />
                      </div>
                      <div>
                        <label className="block text-sm font-medium text-gray-700 mb-1">End Time</label>
                        <input
                          type="time"
                          value={projectInfo.endTime}
                          onChange={(e) => setProjectInfo({...projectInfo, endTime: e.target.value})}
                          className="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
                        />
                      </div>
                    </div>
                  </div>
                </div>
              </div>

              {/* Test Conditions */}
              <div className="mt-6 pt-6 border-t border-gray-200">
                <h3 className="text-lg font-semibold text-gray-700 mb-4">Test Conditions</h3>
                <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">Outdoor Temp (°F)</label>
                    <input
                      type="number"
                      value={projectInfo.outdoorTemp}
                      onChange={(e) => setProjectInfo({...projectInfo, outdoorTemp: e.target.value})}
                      className="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
                      placeholder="°F"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">Indoor Temp (°F)</label>
                    <input
                      type="number"
                      value={projectInfo.indoorTemp}
                      onChange={(e) => setProjectInfo({...projectInfo, indoorTemp: e.target.value})}
                      className="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
                      placeholder="°F"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">Humidity (%)</label>
                    <input
                      type="number"
                      value={projectInfo.humidity}
                      onChange={(e) => setProjectInfo({...projectInfo, humidity: e.target.value})}
                      className="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
                      placeholder="%"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">Weather</label>
                    <select
                      value={projectInfo.weather}
                      onChange={(e) => setProjectInfo({...projectInfo, weather: e.target.value})}
                      className="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
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
          </div>

          {/* Progress Overview */}
          <div className="grid grid-cols-1 md:grid-cols-4 gap-4 mb-8">
            {['split-1', 'split-2', 'split-3', 'split-4'].map((unit) => (
              <div key={unit} className="bg-white rounded-lg shadow-md p-6">
                <h3 className="text-lg font-semibold text-gray-700 capitalize">{unit.replace('-', ' ')}</h3>
                <p className="text-sm text-gray-600 mb-4">
                  {unit === 'split-4' ? '12.5 ton / 20 kW' : '6 ton / 10 kW'}
                </p>
                <div className="w-full bg-gray-200 rounded-full h-2">
                  <div
                    className="bg-blue-600 h-2 rounded-full transition-all duration-300"
                    style={{ width: `${calculateProgress(unit)}%` }}
                  />
                </div>
                <p className="text-xs text-gray-600 mt-2">
                  {Math.round(calculateProgress(unit))}% Complete
                </p>
              </div>
            ))}
          </div>

          {/* Asset Information */}
          <div className="bg-white rounded-lg shadow-md mb-8">
            <div className="bg-gray-100 px-6 py-4 border-b border-gray-200">
              <h2 className="text-xl font-semibold text-gray-800">Asset Information</h2>
            </div>
            <div className="overflow-x-auto">
              <table className="w-full">
                <thead className="bg-gray-50">
                  <tr>
                    <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Field</th>
                    <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Split-1</th>
                    <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Split-2</th>
                    <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Split-3</th>
                    <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Split-4</th>
                  </tr>
                </thead>
                <tbody className="bg-white divide-y divide-gray-200">
                  {['make', 'model', 'serial', 'assetNum', 'serviceArea'].map((field) => (
                    <tr key={field} className="hover:bg-gray-50">
                      <td className="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                        {field === 'assetNum' ? 'Walgreens Asset #' : 
                         field === 'serviceArea' ? 'Service Area' :
                         field.charAt(0).toUpperCase() + field.slice(1)}
                      </td>
                      {['split-1', 'split-2', 'split-3', 'split-4'].map((unit) => (
                        <td key={unit} className="px-6 py-4 whitespace-nowrap">
                          <input
                            type="text"
                            value={assetInfo[unit][field]}
                            onChange={(e) => setAssetInfo({
                              ...assetInfo,
                              [unit]: {...assetInfo[unit], [field]: e.target.value}
                            })}
                            className="w-full px-2 py-1 border border-gray-300 rounded focus:ring-blue-500 focus:border-blue-500 text-sm"
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

          {/* Pre-Functional Checklists */}
          <div className="bg-white rounded-lg shadow-md mb-8">
            <div className="bg-gray-100 px-6 py-4 border-b border-gray-200">
              <h2 className="text-xl font-semibold text-gray-800">Pre-Functional Checklist - Air Handling Units</h2>
            </div>
            <div className="overflow-x-auto">
              <table className="w-full">
                <thead className="bg-gray-50">
                  <tr>
                    <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Inspection Item</th>
                    <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Split-1</th>
                    <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Split-2</th>
                    <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Split-3</th>
                    <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Split-4</th>
                  </tr>
                </thead>
                <tbody className="bg-white divide-y divide-gray-200">
                  {airHandlingChecklist.map((item) => (
                    <tr key={item.id} className="hover:bg-gray-50">
                      <td className="px-6 py-4 text-sm text-gray-900">{item.label}</td>
                      {['split-1', 'split-2', 'split-3', 'split-4'].map((unit) => (
                        <td key={unit} className="px-6 py-4 text-center">
                          <CheckButton
                            value={checklists[`airhandling-${item.id}-${unit}`]}
                            onChange={(value) => updateChecklist(`airhandling-${item.id}`, unit, value)}
                          />
                        </td>
                      ))}
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>

          {/* Functional Test Results */}
          <div className="bg-white rounded-lg shadow-md mb-8">
            <div className="bg-gray-100 px-6 py-4 border-b border-gray-200">
              <h2 className="text-xl font-semibold text-gray-800">Functional Test Results</h2>
            </div>
            <div className="overflow-x-auto">
              <table className="w-full">
                <thead className="bg-gray-50">
                  <tr>
                    <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider w-1/3">Test Description</th>
                    <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Split-1</th>
                    <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Split-2</th>
                    <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Split-3</th>
                    <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Split-4</th>
                  </tr>
                </thead>
                <tbody className="bg-white divide-y divide-gray-200">
                  {functionalTests.map((test) => (
                    <tr key={test.id} className="hover:bg-gray-50">
                      <td className="px-6 py-4">
                        <div>
                          <p className="text-sm font-semibold text-gray-900">{test.name}</p>
                          <p className="text-xs text-gray-600 mt-1">{test.desc}</p>
                          {test.target && (
                            <p className="text-xs text-blue-600 font-mono mt-1">{test.target}</p>
                          )}
                        </div>
                      </td>
                      {['split-1', 'split-2', 'split-3', 'split-4'].map((unit) => (
                        <td key={unit} className="px-6 py-4">
                          <textarea
                            value={testResults[`${test.id}-${unit}`] || ''}
                            onChange={(e) => updateTestResult(test.id, unit, e.target.value)}
                            className="w-full px-2 py-1 border border-gray-300 rounded focus:ring-blue-500 focus:border-blue-500 text-sm font-mono resize-none"
                            rows="2"
                            placeholder="Enter results"
                          />
                        </td>
                      ))}
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>

          {/* Field Notes */}
          <div className="bg-white rounded-lg shadow-md mb-8">
            <div className="bg-gray-100 px-6 py-4 border-b border-gray-200">
              <h2 className="text-xl font-semibold text-gray-800">Field Notes & Issues</h2>
            </div>
            <div className="p-6">
              <div className="bg-yellow-50 border border-yellow-200 rounded-lg p-4">
                <h4 className="text-sm font-semibold text-yellow-800 mb-2 flex items-center gap-2">
                  <AlertCircle className="w-4 h-4" />
                  Test Notes
                </h4>
                <textarea
                  value={fieldNotes}
                  onChange={(e) => setFieldNotes(e.target.value)}
                  className="w-full px-3 py-2 border border-yellow-300 rounded-md focus:ring-yellow-500 focus:border-yellow-500 min-h-[120px] text-sm"
                  placeholder="Record any issues, deviations, or special observations during testing..."
                />
              </div>
            </div>
          </div>

          {/* Acceptance Criteria */}
          <div className="bg-white rounded-lg shadow-md mb-8">
            <div className="bg-gray-100 px-6 py-4 border-b border-gray-200">
              <h2 className="text-xl font-semibold text-gray-800">Acceptance Criteria</h2>
            </div>
            <div className="p-6">
              <div className="space-y-4">
                {[
                  'Cooling capacity within ±5% of 66.3 MBH (6 ton) or 135 MBH (12.5 ton) performance tables',
                  'Electric heater kW within ±10% of nameplate (10 / 20 kW)',
                  'Steam coil leaving air temp reaches ≥ 90°F within 3 min and stable ±2°F',
                  'Economizer provides at least 25% OA at minimum position and maintains T_mix ≤ 55°F without compressors when OAT ≤ 55°F',
                  'All safety shutdowns and alarms annunciated at BAS and local unit controller'
                ].map((criteria, index) => (
                  <div key={index} className="flex items-start gap-3">
                    <input
                      type="checkbox"
                      className="mt-1 w-5 h-5 text-blue-600 border-gray-300 rounded focus:ring-blue-500"
                    />
                    <label className="text-sm text-gray-700">{criteria}</label>
                  </div>
                ))}
              </div>
            </div>
          </div>

          {/* Signatures Section */}
          <div className="bg-white rounded-lg shadow-md">
            <div className="bg-blue-900 text-white px-6 py-4 rounded-t-lg">
              <h2 className="text-xl font-semibold">Certification & Signatures</h2>
            </div>
            <div className="p-6">
              <p className="text-sm text-gray-700 mb-6">
                By signing below, all parties certify that the equipment has been installed, tested, and commissioned in accordance with the manufacturer's specifications, applicable codes, and the requirements outlined in this commissioning test script.
              </p>
              
              <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                {/* Installing Contractor */}
                <div className="border border-gray-200 rounded-lg p-6">
                  <h4 className="text-sm font-semibold text-gray-700 mb-4">Installing Contractor</h4>
                  <div className="space-y-4">
                    <div>
                      <label className="block text-xs text-gray-600 mb-1">Print Name & Title</label>
                      <input
                        type="text"
                        className="w-full px-3 py-2 border-b border-gray-300 focus:border-blue-500 outline-none"
                      />
                    </div>
                    <div>
                      <label className="block text-xs text-gray-600 mb-1">Date</label>
                      <input
                        type="date"
                        className="w-full px-3 py-2 border-b border-gray-300 focus:border-blue-500 outline-none"
                      />
                    </div>
                    <div>
                      <label className="block text-xs text-gray-600 mb-1">Signature</label>
                      <div className="h-16 border-b border-gray-300"></div>
                    </div>
                  </div>
                </div>

                {/* Commissioning Agent */}
                <div className="border border-gray-200 rounded-lg p-6">
                  <h4 className="text-sm font-semibold text-gray-700 mb-4">Commissioning Agent</h4>
                  <div className="space-y-4">
                    <div>
                      <label className="block text-xs text-gray-600 mb-1">Print Name & Title</label>
                      <input
                        type="text"
                        className="w-full px-3 py-2 border-b border-gray-300 focus:border-blue-500 outline-none"
                      />
                    </div>
                    <div>
                      <label className="block text-xs text-gray-600 mb-1">Date</label>
                      <input
                        type="date"
                        className="w-full px-3 py-2 border-b border-gray-300 focus:border-blue-500 outline-none"
                      />
                    </div>
                    <div>
                      <label className="block text-xs text-gray-600 mb-1">Signature</label>
                      <div className="h-16 border-b border-gray-300"></div>
                    </div>
                  </div>
                </div>
              </div>

              {/* Walgreens Representative */}
              <div className="mt-6 border border-gray-200 rounded-lg p-6">
                <h4 className="text-sm font-semibold text-gray-700 mb-4">Walgreens Representative</h4>
                <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                  <div className="space-y-4">
                    <div>
                      <label className="block text-xs text-gray-600 mb-1">Print Name & Title</label>
                      <input
                        type="text"
                        className="w-full px-3 py-2 border-b border-gray-300 focus:border-blue-500 outline-none"
                      />
                    </div>
                    <div>
                      <label className="block text-xs text-gray-600 mb-1">Date</label>
                      <input
                        type="date"
                        className="w-full px-3 py-2 border-b border-gray-300 focus:border-blue-500 outline-none"
                      />
                    </div>
                  </div>
                  <div className="space-y-4">
                    <div>
                      <label className="block text-xs text-gray-600 mb-1">Signature</label>
                      <div className="h-16 border-b border-gray-300"></div>
                    </div>
                    <div>
                      <label className="block text-xs text-gray-600 mb-2">Acceptance Status:</label>
                      <div className="flex gap-4">
                        <label className="flex items-center gap-2 text-sm">
                          <input type="radio" name="acceptance" className="text-blue-600" />
                          Accepted
                        </label>
                        <label className="flex items-center gap-2 text-sm">
                          <input type="radio" name="acceptance" className="text-blue-600" />
                          Conditional
                        </label>
                        <label className="flex items-center gap-2 text-sm">
                          <input type="radio" name="acceptance" className="text-blue-600" />
                          Rejected
                        </label>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </>
  );
};

export default CommissioningTestScript;
