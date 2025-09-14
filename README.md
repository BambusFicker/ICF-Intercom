import React, { useState, useRef, useEffect } from 'react';
import { Mic, MicOff, Volume2, VolumeX, Users, Settings, Phone, PhoneOff, MessageSquare, Radio } from 'lucide-react';

const TeamIntercomApp = () => {
  const [isRecording, setIsRecording] = useState(false);
  const [isMuted, setIsMuted] = useState(false);
  const [isConnected, setIsConnected] = useState(false);
  const [selectedChannel, setSelectedChannel] = useState('general');
  const [activeUsers, setActiveUsers] = useState([
    { id: 1, name: 'Alex', status: 'online', speaking: false },
    { id: 2, name: 'Sarah', status: 'online', speaking: false },
    { id: 3, name: 'Mike', status: 'away', speaking: false },
    { id: 4, name: 'Emma', status: 'online', speaking: true }
  ]);
  const [messages, setMessages] = useState([
    { id: 1, user: 'Sarah', message: 'Team meeting in 5 minutes', time: '14:23', type: 'text' },
    { id: 2, user: 'Alex', message: 'Voice message received', time: '14:25', type: 'voice' }
  ]);
  const [currentMessage, setCurrentMessage] = useState('');
  const [channels] = useState([
    { id: 'general', name: 'General', users: 4 },
    { id: 'dev-team', name: 'Dev Team', users: 3 },
    { id: 'management', name: 'Management', users: 2 },
    { id: 'support', name: 'Support', users: 5 }
  ]);

  const audioRef = useRef(null);
  const [recordingTime, setRecordingTime] = useState(0);
  const [volume, setVolume] = useState(75);

  useEffect(() => {
    let interval;
    if (isRecording) {
      interval = setInterval(() => {
        setRecordingTime(prev => prev + 1);
      }, 1000);
    } else {
      setRecordingTime(0);
    }
    return () => clearInterval(interval);
  }, [isRecording]);

  const toggleRecording = () => {
    setIsRecording(!isRecording);
    if (!isRecording) {
      // Simulate starting recording
      navigator.mediaDevices?.getUserMedia({ audio: true })
        .then(stream => {
          console.log('Recording started');
        })
        .catch(err => console.log('Microphone access denied'));
    }
  };

  const toggleConnection = () => {
    setIsConnected(!isConnected);
  };

  const sendMessage = () => {
    if (currentMessage.trim()) {
      const newMessage = {
        id: messages.length + 1,
        user: 'You',
        message: currentMessage,
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
        type: 'text'
      };
      setMessages([...messages, newMessage]);
      setCurrentMessage('');
    }
  };

  const formatTime = (seconds) => {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return `${mins}:${secs.toString().padStart(2, '0')}`;
  };

  return (
    <div className="flex h-screen bg-gray-900 text-white">
      {/* Sidebar */}
      <div className="w-80 bg-gray-800 border-r border-gray-700 flex flex-col">
        {/* Header */}
        <div className="p-4 border-b border-gray-700">
          <div className="flex items-center justify-between">
            <h1 className="text-xl font-bold flex items-center gap-2">
              <Radio className="w-6 h-6 text-blue-500" />
              Team Intercom
            </h1>
            <Settings className="w-5 h-5 text-gray-400 cursor-pointer hover:text-white" />
          </div>
        </div>

        {/* Channels */}
        <div className="flex-1 p-4">
          <h2 className="text-sm font-semibold text-gray-400 mb-3">CHANNELS</h2>
          <div className="space-y-2">
            {channels.map(channel => (
              <div
                key={channel.id}
                onClick={() => setSelectedChannel(channel.id)}
                className={`p-3 rounded-lg cursor-pointer transition-colors ${
                  selectedChannel === channel.id 
                    ? 'bg-blue-600 text-white' 
                    : 'bg-gray-700 hover:bg-gray-600'
                }`}
              >
                <div className="flex items-center justify-between">
                  <span className="font-medium"># {channel.name}</span>
                  <span className="text-xs text-gray-300">{channel.users}</span>
                </div>
              </div>
            ))}
          </div>

          {/* Active Users */}
          <h2 className="text-sm font-semibold text-gray-400 mb-3 mt-6">ACTIVE USERS</h2>
          <div className="space-y-2">
            {activeUsers.map(user => (
              <div key={user.id} className="flex items-center gap-3 p-2 rounded-lg">
                <div className={`w-3 h-3 rounded-full ${
                  user.status === 'online' ? 'bg-green-500' : 
                  user.status === 'away' ? 'bg-yellow-500' : 'bg-gray-500'
                }`} />
                <span className="text-sm">{user.name}</span>
                {user.speaking && (
                  <Volume2 className="w-4 h-4 text-green-500 animate-pulse" />
                )}
              </div>
            ))}
          </div>
        </div>
      </div>

      {/* Main Content */}
      <div className="flex-1 flex flex-col">
        {/* Channel Header */}
        <div className="p-4 bg-gray-800 border-b border-gray-700">
          <div className="flex items-center justify-between">
            <div>
              <h2 className="text-lg font-semibold"># {channels.find(c => c.id === selectedChannel)?.name}</h2>
              <p className="text-sm text-gray-400">
                {activeUsers.filter(u => u.status === 'online').length} online members
              </p>
            </div>
            <div className="flex items-center gap-4">
              <button
                onClick={toggleConnection}
                className={`p-2 rounded-lg transition-colors ${
                  isConnected 
                    ? 'bg-green-600 hover:bg-green-700' 
                    : 'bg-gray-600 hover:bg-gray-700'
                }`}
              >
                {isConnected ? <Phone className="w-5 h-5" /> : <PhoneOff className="w-5 h-5" />}
              </button>
              <Users className="w-5 h-5 text-gray-400" />
            </div>
          </div>
        </div>

        {/* Messages Area */}
        <div className="flex-1 p-4 overflow-y-auto space-y-4">
          {messages.map(message => (
            <div key={message.id} className="flex items-start gap-3">
              <div className="w-8 h-8 bg-blue-600 rounded-full flex items-center justify-center text-sm font-medium">
                {message.user[0]}
              </div>
              <div>
                <div className="flex items-center gap-2 mb-1">
                  <span className="font-medium">{message.user}</span>
                  <span className="text-xs text-gray-400">{message.time}</span>
                </div>
                <div className={`p-3 rounded-lg max-w-md ${
                  message.type === 'voice' 
                    ? 'bg-blue-600' 
                    : 'bg-gray-700'
                }`}>
                  {message.type === 'voice' ? (
                    <div className="flex items-center gap-2">
                      <Volume2 className="w-4 h-4" />
                      <span className="text-sm">Voice Message (0:15)</span>
                    </div>
                  ) : (
                    <p>{message.message}</p>
                  )}
                </div>
              </div>
            </div>
          ))}
        </div>

        {/* Input Area */}
        <div className="p-4 bg-gray-800 border-t border-gray-700">
          <div className="flex items-center gap-4">
            <div className="flex-1 flex items-center gap-2 bg-gray-700 rounded-lg p-2">
              <input
                type="text"
                value={currentMessage}
                onChange={(e) => setCurrentMessage(e.target.value)}
                onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
                placeholder="Type a message..."
                className="flex-1 bg-transparent outline-none"
              />
              <button
                onClick={sendMessage}
                className="p-2 bg-blue-600 rounded-lg hover:bg-blue-700 transition-colors"
              >
                <MessageSquare className="w-4 h-4" />
              </button>
            </div>
          </div>
        </div>
      </div>

      {/* Audio Controls Panel */}
      <div className="w-72 bg-gray-800 border-l border-gray-700 p-4">
        <h3 className="text-lg font-semibold mb-4">Audio Controls</h3>
        
        {/* Push to Talk */}
        <div className="mb-6">
          <div className="flex items-center justify-between mb-2">
            <span className="text-sm font-medium">Push to Talk</span>
            {isRecording && (
              <span className="text-xs text-red-500">{formatTime(recordingTime)}</span>
            )}
          </div>
          <button
            onClick={toggleRecording}
            onMouseDown={() => setIsRecording(true)}
            onMouseUp={() => setIsRecording(false)}
            onMouseLeave={() => setIsRecording(false)}
            className={`w-full p-4 rounded-lg border-2 transition-all ${
              isRecording 
                ? 'bg-red-600 border-red-500 animate-pulse' 
                : 'bg-gray-700 border-gray-600 hover:border-gray-500'
            }`}
          >
            {isRecording ? (
              <div className="flex flex-col items-center gap-2">
                <Mic className="w-8 h-8 text-white" />
                <span className="text-sm font-medium">RECORDING</span>
              </div>
            ) : (
              <div className="flex flex-col items-center gap-2">
                <MicOff className="w-8 h-8 text-gray-400" />
                <span className="text-sm">Hold to Talk</span>
              </div>
            )}
          </button>
          <p className="text-xs text-gray-400 mt-2 text-center">
            Hold button or spacebar to transmit
          </p>
        </div>

        {/* Volume Control */}
        <div className="mb-6">
          <div className="flex items-center justify-between mb-2">
            <span className="text-sm font-medium">Volume</span>
            <span className="text-xs text-gray-400">{volume}%</span>
          </div>
          <div className="flex items-center gap-3">
            <button
              onClick={() => setIsMuted(!isMuted)}
              className="p-2 bg-gray-700 rounded-lg hover:bg-gray-600"
            >
              {isMuted ? <VolumeX className="w-4 h-4" /> : <Volume2 className="w-4 h-4" />}
            </button>
            <input
              type="range"
              min="0"
              max="100"
              value={volume}
              onChange={(e) => setVolume(e.target.value)}
              className="flex-1 h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer"
            />
          </div>
        </div>

        {/* Connection Status */}
        <div className="p-4 bg-gray-700 rounded-lg">
          <div className="flex items-center gap-2 mb-2">
            <div className={`w-3 h-3 rounded-full ${isConnected ? 'bg-green-500' : 'bg-red-500'}`} />
            <span className="text-sm font-medium">
              {isConnected ? 'Connected' : 'Disconnected'}
            </span>
          </div>
          <p className="text-xs text-gray-400">
            {isConnected ? 'Audio channel active' : 'Click to connect to voice channel'}
          </p>
        </div>

        {/* Quick Actions */}
        <div className="mt-6">
          <h4 className="text-sm font-medium mb-3">Quick Actions</h4>
          <div className="space-y-2">
            <button className="w-full p-2 text-left text-sm bg-gray-700 rounded-lg hover:bg-gray-600">
              🔊 Broadcast to All
            </button>
            <button className="w-full p-2 text-left text-sm bg-gray-700 rounded-lg hover:bg-gray-600">
              📢 Emergency Alert
            </button>
            <button className="w-full p-2 text-left text-sm bg-gray-700 rounded-lg hover:bg-gray-600">
              🎵 Play Alert Sound
            </button>
          </div>
        </div>
      </div>
    </div>
  );
};

export default TeamIntercomApp;
