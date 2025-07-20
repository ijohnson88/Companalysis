import React, { useState, useEffect, useRef } from 'react';
import { Mic, MicOff, Settings, AlertTriangle, TrendingUp, DollarSign, Users, Zap, AlertCircle, CheckCircle } from 'lucide-react';

const CompetitorIntelligenceApp = () => {
  const [isListening, setIsListening] = useState(false);
  const [transcript, setTranscript] = useState('');
  const [detectedCompetitors, setDetectedCompetitors] = useState([]);
  const [userCompany, setUserCompany] = useState('');
  const [isSetupOpen, setIsSetupOpen] = useState(false);
  const [competitorIntel, setCompetitorIntel] = useState(null);
  const [isProcessing, setIsProcessing] = useState(false);
  const [speechSupported, setSpeechSupported] = useState(false);
  const [speechError, setSpeechError] = useState('');
  const [manualInput, setManualInput] = useState('');
  const recognitionRef = useRef(null);

  // Sample competitor database
  const competitorDatabase = {
    'CRM': {
      'salesforce': {
        name: 'Salesforce',
        strengths: ['Market leader', 'Extensive ecosystem', 'Advanced AI features'],
        weaknesses: ['Complex interface', 'Expensive for small teams', 'Steep learning curve'],
        pricing: '$25-300/user/month',
        marketShare: '19.5%'
      },
      'hubspot': {
        name: 'HubSpot',
        strengths: ['Easy to use', 'Free tier available', 'All-in-one platform'],
        weaknesses: ['Limited customization', 'Expensive at scale', 'Basic reporting'],
        pricing: '$0-3,200/month',
        marketShare: '8.8%'
      },
      'pipedrive': {
        name: 'Pipedrive',
        strengths: ['Visual pipeline', 'Simple interface', 'Affordable pricing'],
        weaknesses: ['Limited features', 'Basic automation', 'No marketing tools'],
        pricing: '$14-99/user/month',
        marketShare: '3.2%'
      }
    },
    'Communication': {
      'slack': {
        name: 'Slack',
        strengths: ['Superior UX', 'App ecosystem', 'Search functionality'],
        weaknesses: ['No video conferencing', 'Thread management', 'Pricing complexity'],
        pricing: '$0-24/user/month',
        marketShare: '12.1%'
      },
      'teams': {
        name: 'Microsoft Teams',
        strengths: ['Office 365 integration', 'Video conferencing', 'Enterprise features'],
        weaknesses: ['Slower performance', 'Complex interface', 'Resource heavy'],
        pricing: '$4-22/user/month',
        marketShare: '44.2%'
      },
      'discord': {
        name: 'Discord',
        strengths: ['Great for communities', 'Voice quality', 'Gaming features'],
        weaknesses: ['Not business-focused', 'Limited enterprise features', 'Security concerns'],
        pricing: '$0-15/user/month',
        marketShare: '2.1%'
      }
    },
    'DevOps': {
      'datadog': {
        name: 'Datadog',
        strengths: ['Unified platform', 'Great visualizations', 'Machine learning'],
        weaknesses: ['Expensive at scale', 'Complex pricing', 'Learning curve'],
        pricing: '$15-23/host/month',
        marketShare: '8.7%'
      },
      'newrelic': {
        name: 'New Relic',
        strengths: ['APM leader', 'Real user monitoring', 'Browser insights'],
        weaknesses: ['Pricing complexity', 'Limited infrastructure', 'Data retention'],
        pricing: '$25-750/month',
        marketShare: '5.4%'
      }
    }
  };

  // Company value propositions
  const companyProfiles = {
    'Salesforce': {
      industry: 'CRM',
      valueProps: ['Enterprise-grade security', 'Extensive customization', 'Proven scalability'],
      competitors: ['hubspot', 'pipedrive', 'zoho']
    },
    'HubSpot': {
      industry: 'CRM',
      valueProps: ['All-in-one platform', 'Easy setup', 'Free tier'],
      competitors: ['salesforce', 'pipedrive', 'marketo']
    },
    'Slack': {
      industry: 'Communication',
      valueProps: ['Superior UX', 'App integrations', 'Developer-friendly'],
      competitors: ['teams', 'discord', 'zoom']
    },
    'Microsoft': {
      industry: 'Communication',
      valueProps: ['Office 365 integration', 'Enterprise security', 'Video conferencing'],
      competitors: ['slack', 'zoom', 'google workspace']
    },
    'Datadog': {
      industry: 'DevOps',
      valueProps: ['Full-stack monitoring', 'Machine learning insights', 'Unified dashboards'],
      competitors: ['newrelic', 'splunk', 'dynatrace']
    }
  };

  // Check for speech recognition support
  useEffect(() => {
    if ('webkitSpeechRecognition' in window || 'SpeechRecognition' in window) {
      setSpeechSupported(true);
      
      const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
      const recognition = new SpeechRecognition();
      recognition.continuous = true;
      recognition.interimResults = true;
      recognition.lang = 'en-US';
      
      recognition.onstart = () => {
        setSpeechError('');
        console.log('Speech recognition started');
      };
      
      recognition.onresult = (event) => {
        let currentTranscript = '';
        for (let i = event.resultIndex; i < event.results.length; i++) {
          if (event.results[i].isFinal) {
            currentTranscript += event.results[i][0].transcript;
          }
        }
        if (currentTranscript) {
          setTranscript(prev => prev + ' ' + currentTranscript);
          detectCompetitors(currentTranscript);
        }
      };

      recognition.onerror = (event) => {
        console.error('Speech recognition error:', event.error);
        setSpeechError(`Speech recognition error: ${event.error}`);
        setIsListening(false);
        
        if (event.error === 'not-allowed') {
          setSpeechError('Microphone access denied. Please allow microphone permissions and try again.');
        } else if (event.error === 'no-speech') {
          setSpeechError('No speech detected. Please try speaking louder.');
        }
      };

      recognition.onend = () => {
        console.log('Speech recognition ended');
        setIsListening(false);
      };

      recognitionRef.current = recognition;
    } else {
      setSpeechSupported(false);
      setSpeechError('Speech recognition not supported in this browser. Try Chrome or Edge.');
    }
  }, []);

  const detectCompetitors = (text) => {
    const lowerText = text.toLowerCase();
    const userProfile = companyProfiles[userCompany];
    
    if (!userProfile) return;

    const industryCompetitors = competitorDatabase[userProfile.industry] || {};
    
    Object.keys(industryCompetitors).forEach(competitorKey => {
      const competitor = industryCompetitors[competitorKey];
      const competitorName = competitor.name.toLowerCase();
      
      if (lowerText.includes(competitorName) || lowerText.includes(competitorKey)) {
        setDetectedCompetitors(prev => {
          const existing = prev.find(c => c.name === competitor.name);
          if (!existing) {
            generateCompetitorIntel(competitor, userProfile);
            return [...prev, { ...competitor, timestamp: new Date().toLocaleTimeString() }];
          }
          return prev;
        });
      }
    });
  };

  const generateCompetitorIntel = (competitor, userProfile) => {
    setIsProcessing(true);
    
    // Simulate AI processing delay
    setTimeout(() => {
      const intel = {
        competitor: competitor.name,
        theirStrengths: competitor.strengths,
        theirWeaknesses: competitor.weaknesses,
        pricing: competitor.pricing,
        marketShare: competitor.marketShare,
        yourAdvantages: userProfile.valueProps,
        talkingPoints: generateTalkingPoints(competitor, userProfile),
        battlecard: generateBattlecard(competitor, userProfile)
      };
      
      setCompetitorIntel(intel);
      setIsProcessing(false);
    }, 800);
  };

  const generateTalkingPoints = (competitor, userProfile) => {
    const points = [
      `While ${competitor.name} ${competitor.strengths[0].toLowerCase()}, our ${userProfile.valueProps[0].toLowerCase()} provides superior value for enterprise customers.`,
      `${competitor.name}'s main challenge is ${competitor.weaknesses[0].toLowerCase()}. We solve this with ${userProfile.valueProps[1].toLowerCase()}.`,
      `Consider the total cost of ownership - our ${userProfile.valueProps[2] ? userProfile.valueProps[2].toLowerCase() : 'proven ROI'} often results in 30-50% cost savings over time.`
    ];
    return points;
  };

  const generateBattlecard = (competitor, userProfile) => {
    return {
      whenToUse: `When prospect mentions ${competitor.name} or shows interest in ${competitor.strengths[0].toLowerCase()}`,
      keyMessage: `"${competitor.name} is a solid choice, but let me show you how we specifically address ${competitor.weaknesses[0].toLowerCase()} while providing ${userProfile.valueProps[0].toLowerCase()}."`,
      objectionHandling: {
        price: `"While ${competitor.name} may appear less expensive upfront, our customers typically see 40% better ROI due to ${userProfile.valueProps[0].toLowerCase()}."`,
        features: `"${competitor.name} ${competitor.strengths[0].toLowerCase()}, but they struggle with ${competitor.weaknesses[0].toLowerCase()}. Here's how we solve that..."`,
        brand: `"${competitor.name} has strong brand recognition, but ${userProfile.valueProps[1] ? userProfile.valueProps[1].toLowerCase() : 'our innovation'} is what drives actual business results."`
      }
    };
  };

  const toggleListening = () => {
    if (!speechSupported) {
      setSpeechError('Speech recognition not supported. Use manual input below.');
      return;
    }

    if (isListening) {
      recognitionRef.current?.stop();
      setIsListening(false);
    } else {
      if (!userCompany) {
        setIsSetupOpen(true);
        return;
      }
      setSpeechError('');
      recognitionRef.current?.start();
      setIsListening(true);
    }
  };

  const handleManualInput = (e) => {
    e.preventDefault();
    if (manualInput.trim()) {
      setTranscript(prev => prev + ' ' + manualInput);
      detectCompetitors(manualInput);
      setManualInput('');
    }
  };

  const clearSession = () => {
    setTranscript('');
    setDetectedCompetitors([]);
    setCompetitorIntel(null);
    setSpeechError('');
  };

  const simulateCompetitorMention = () => {
    const sampleTexts = [
      "We're currently using Salesforce but it's quite expensive",
      "The team loves Slack but we need better video conferencing",
      "We've been looking at HubSpot for our marketing automation",
      "Our monitoring setup uses New Relic but it's getting complex",
      "Microsoft Teams comes with our Office license"
    ];
    
    const randomText = sampleTexts[Math.floor(Math.random() * sampleTexts.length)];
    setTranscript(prev => prev + ' ' + randomText);
    detectCompetitors(randomText);
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-4">
      <div className="max-w-7xl mx-auto">
        {/* Header */}
        <div className="bg-white rounded-xl shadow-lg p-6 mb-6">
          <div className="flex items-center justify-between">
            <div>
              <h1 className="text-3xl font-bold text-gray-900 mb-2">
                🎯 Competitive Intelligence AI
              </h1>
              <p className="text-gray-600">
                Real-time competitor analysis for {userCompany || 'your sales calls'}
              </p>
              
              {/* Status Indicators */}
              <div className="flex items-center gap-4 mt-3">
                <div className="flex items-center gap-2">
                  {speechSupported ? (
                    <CheckCircle className="w-4 h-4 text-green-500" />
                  ) : (
                    <AlertCircle className="w-4 h-4 text-red-500" />
                  )}
                  <span className="text-sm text-gray-600">
                    Speech Recognition: {speechSupported ? 'Supported' : 'Not Supported'}
                  </span>
                </div>
                {userCompany && (
                  <div className="flex items-center gap-2">
                    <CheckCircle className="w-4 h-4 text-green-500" />
                    <span className="text-sm text-gray-600">Company: {userCompany}</span>
                  </div>
                )}
              </div>
            </div>
            <div className="flex gap-3">
              <button
                onClick={() => setIsSetupOpen(true)}
                className="flex items-center gap-2 px-4 py-2 bg-gray-100 hover:bg-gray-200 rounded-lg transition-colors"
              >
                <Settings className="w-4 h-4" />
                Setup
              </button>
              <button
                onClick={simulateCompetitorMention}
                className="px-4 py-2 bg-indigo-100 hover:bg-indigo-200 text-indigo-700 rounded-lg transition-colors"
              >
                Demo Competitor Mention
              </button>
            </div>
          </div>
        </div>

        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
          {/* Left Column - Audio Input & Transcript */}
          <div className="lg:col-span-1">
            <div className="bg-white rounded-xl shadow-lg p-6 mb-6">
              <h2 className="text-xl font-semibold mb-4">Live Audio Input</h2>
              
              {speechError && (
                <div className="mb-4 p-3 bg-red-50 border border-red-200 rounded-lg">
                  <div className="flex items-center gap-2">
                    <AlertCircle className="w-4 h-4 text-red-500" />
                    <span className="text-sm text-red-700">{speechError}</span>
                  </div>
                </div>
              )}
              
              <div className="text-center mb-4">
                <button
                  onClick={toggleListening}
                  disabled={!speechSupported || !userCompany}
                  className={`w-20 h-20 rounded-full flex items-center justify-center text-white text-2xl transition-all duration-300 ${
                    !speechSupported || !userCompany
                      ? 'bg-gray-400 cursor-not-allowed'
                      : isListening 
                      ? 'bg-red-500 hover:bg-red-600 animate-pulse' 
                      : 'bg-blue-500 hover:bg-blue-600'
                  }`}
                >
                  {isListening ? <MicOff /> : <Mic />}
                </button>
                <p className="mt-2 text-sm text-gray-600">
                  {!userCompany 
                    ? 'Setup company first' 
                    : !speechSupported 
                    ? 'Speech not supported' 
                    : isListening 
                    ? 'Listening...' 
                    : 'Click to start listening'}
                </p>
              </div>

              {/* Manual Input Fallback */}
              <div className="border-t pt-4">
                <h3 className="font-medium mb-2">Manual Input</h3>
                <form onSubmit={handleManualInput}>
                  <div className="flex gap-2">
                    <input
                      type="text"
                      value={manualInput}
                      onChange={(e) => setManualInput(e.target.value)}
                      placeholder="Type competitor mention..."
                      className="flex-1 border border-gray-300 rounded px-3 py-2 text-sm"
                    />
                    <button
                      type="submit"
                      disabled={!userCompany}
                      className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 disabled:bg-gray-400"
                    >
                      Add
                    </button>
                  </div>
                </form>
              </div>

              {transcript && (
                <div className="border rounded-lg p-4 bg-gray-50 mt-4">
                  <h3 className="font-medium mb-2">Transcript:</h3>
                  <p className="text-sm text-gray-700">{transcript}</p>
                  <button
                    onClick={clearSession}
                    className="mt-2 text-xs text-blue-600 hover:text-blue-800"
                  >
                    Clear Session
                  </button>
                </div>
              )}
            </div>

            {/* Detected Competitors */}
            {detectedCompetitors.length > 0 && (
              <div className="bg-white rounded-xl shadow-lg p-6">
                <h2 className="text-xl font-semibold mb-4 flex items-center gap-2">
                  <AlertTriangle className="w-5 h-5 text-orange-500" />
                  Competitors Detected
                </h2>
                <div className="space-y-3">
                  {detectedCompetitors.map((competitor, index) => (
                    <div key={index} className="border-l-4 border-orange-500 pl-4 py-2 bg-orange-50 rounded-r-lg">
                      <div className="flex justify-between items-start">
                        <div>
                          <h3 className="font-medium text-orange-900">{competitor.name}</h3>
                          <p className="text-sm text-orange-700">Detected at {competitor.timestamp}</p>
                        </div>
                        <span className="bg-orange-200 text-orange-800 px-2 py-1 rounded text-xs">
                          Active
                        </span>
                      </div>
                    </div>
                  ))}
                </div>
              </div>
            )}
          </div>

          {/* Right Column - Competitive Intelligence */}
          <div className="lg:col-span-2">
            {isProcessing && (
              <div className="bg-white rounded-xl shadow-lg p-6 mb-6">
                <div className="flex items-center gap-3">
                  <div className="animate-spin rounded-full h-6 w-6 border-b-2 border-blue-500"></div>
                  <span>Generating competitive intelligence...</span>
                </div>
              </div>
            )}

            {competitorIntel && (
              <div className="space-y-6">
                {/* Quick Stats */}
                <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
                  <div className="bg-white rounded-lg shadow p-4">
                    <div className="flex items-center gap-2 mb-2">
                      <TrendingUp className="w-4 h-4 text-green-500" />
                      <span className="text-sm font-medium">Market Share</span>
                    </div>
                    <p className="text-xl font-bold text-gray-900">{competitorIntel.marketShare}</p>
                  </div>
                  <div className="bg-white rounded-lg shadow p-4">
                    <div className="flex items-center gap-2 mb-2">
                      <DollarSign className="w-4 h-4 text-blue-500" />
                      <span className="text-sm font-medium">Pricing</span>
                    </div>
                    <p className="text-sm font-bold text-gray-900">{competitorIntel.pricing}</p>
                  </div>
                  <div className="bg-white rounded-lg shadow p-4">
                    <div className="flex items-center gap-2 mb-2">
                      <Zap className="w-4 h-4 text-yellow-500" />
                      <span className="text-sm font-medium">Strengths</span>
                    </div>
                    <p className="text-sm font-bold text-gray-900">{competitorIntel.theirStrengths?.length || 0}</p>
                  </div>
                  <div className="bg-white rounded-lg shadow p-4">
                    <div className="flex items-center gap-2 mb-2">
                      <Users className="w-4 h-4 text-purple-500" />
                      <span className="text-sm font-medium">Response Time</span>
                    </div>
                    <p className="text-xl font-bold text-gray-900">0.8s</p>
                  </div>
                </div>

                {/* Competitive Analysis */}
                <div className="bg-white rounded-xl shadow-lg p-6">
                  <h2 className="text-xl font-semibold mb-4">
                    🏆 {competitorIntel.competitor} Analysis
                  </h2>
                  
                  <div className="grid md:grid-cols-2 gap-6">
                    <div>
                      <h3 className="font-medium text-red-600 mb-3">Their Strengths</h3>
                      <ul className="space-y-2">
                        {competitorIntel.theirStrengths?.map((strength, index) => (
                          <li key={index} className="text-sm bg-red-50 p-2 rounded border-l-2 border-red-200">
                            {strength}
                          </li>
                        ))}
                      </ul>
                    </div>
                    
                    <div>
                      <h3 className="font-medium text-green-600 mb-3">Your Advantages</h3>
                      <ul className="space-y-2">
                        {competitorIntel.yourAdvantages?.map((advantage, index) => (
                          <li key={index} className="text-sm bg-green-50 p-2 rounded border-l-2 border-green-200">
                            {advantage}
                          </li>
                        ))}
                      </ul>
                    </div>
                  </div>

                  <div className="mt-6">
                    <h3 className="font-medium text-blue-600 mb-3">Their Weaknesses</h3>
                    <ul className="space-y-2">
                      {competitorIntel.theirWeaknesses?.map((weakness, index) => (
                        <li key={index} className="text-sm bg-blue-50 p-2 rounded border-l-2 border-blue-200">
                          {weakness}
                        </li>
                      ))}
                    </ul>
                  </div>
                </div>

                {/* Talking Points */}
                <div className="bg-white rounded-xl shadow-lg p-6">
                  <h2 className="text-xl font-semibold mb-4">💬 Suggested Talking Points</h2>
                  <div className="space-y-4">
                    {competitorIntel.talkingPoints?.map((point, index) => (
                      <div key={index} className="border-l-4 border-blue-500 pl-4 py-2 bg-blue-50 rounded-r-lg">
                        <p className="text-sm text-gray-700">{point}</p>
                      </div>
                    ))}
                  </div>
                </div>

                {/* Battle Card */}
                {competitorIntel.battlecard && (
                  <div className="bg-white rounded-xl shadow-lg p-6">
                    <h2 className="text-xl font-semibold mb-4">⚔️ Battle Card</h2>
                    
                    <div className="space-y-4">
                      <div>
                        <h3 className="font-medium text-gray-900 mb-2">When to Use:</h3>
                        <p className="text-sm text-gray-600 bg-gray-50 p-3 rounded">
                          {competitorIntel.battlecard.whenToUse}
                        </p>
                      </div>
                      
                      <div>
                        <h3 className="font-medium text-gray-900 mb-2">Key Message:</h3>
                        <p className="text-sm text-gray-600 bg-green-50 p-3 rounded border-l-2 border-green-200">
                          {competitorIntel.battlecard.keyMessage}
                        </p>
                      </div>
                      
                      <div>
                        <h3 className="font-medium text-gray-900 mb-2">Objection Handling:</h3>
                        <div className="space-y-2">
                          <div className="bg-yellow-50 p-3 rounded">
                            <strong className="text-yellow-800">Price Objection:</strong>
                            <p className="text-sm text-yellow-700 mt-1">
                              {competitorIntel.battlecard.objectionHandling?.price}
                            </p>
                          </div>
                          <div className="bg-purple-50 p-3 rounded">
                            <strong className="text-purple-800">Feature Objection:</strong>
                            <p className="text-sm text-purple-700 mt-1">
                              {competitorIntel.battlecard.objectionHandling?.features}
                            </p>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                )}
              </div>
            )}
          </div>
        </div>

        {/* Setup Modal */}
        {isSetupOpen && (
          <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
            <div className="bg-white rounded-xl shadow-xl p-6 w-full max-w-md">
              <h2 className="text-xl font-semibold mb-4">Company Setup</h2>
              
              <div className="space-y-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-2">
                    Your Company
                  </label>
                  <select
                    value={userCompany}
                    onChange={(e) => setUserCompany(e.target.value)}
                    className="w-full border border-gray-300 rounded-lg px-3 py-2"
                  >
                    <option value="">Select your company...</option>
                    <option value="Salesforce">Salesforce</option>
                    <option value="HubSpot">HubSpot</option>
                    <option value="Slack">Slack</option>
                    <option value="Microsoft">Microsoft</option>
                    <option value="Datadog">Datadog</option>
                  </select>
                </div>
              </div>
              
              <div className="flex gap-3 mt-6">
                <button
                  onClick={() => setIsSetupOpen(false)}
                  className="flex-1 px-4 py-2 border border-gray-300 rounded-lg hover:bg-gray-50"
                >
                  Cancel
                </button>
                <button
                  onClick={() => {
                    setIsSetupOpen