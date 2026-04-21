import React, { useState, useEffect } from 'react';
import { Shield, Activity, Search, AlertTriangle, Users, Terminal, WifiOff, RefreshCw } from 'lucide-react';

// --- CONFIGURATION ---
// ⚠️ REPLACE 'YOUR_WHIPSBYTE_IP' with the actual IP address provided by Whipsbyte
// ⚠️ Ensure your Whipsbyte Firewall allows traffic on Port 5000
// ⚠️ If using HTTPS for this site, your API must also be HTTPS (or browser will block it)
const API_BASE_URL = "http://YOUR_WHIPSBYTE_IP:5000";

const App = () => {
  const [stats, setStats] = useState({ totalScanned: 0, threatsDetected: 0 });
  const [recentStrikes, setRecentStrikes] = useState([]);
  const [loading, setLoading] = useState(true);
  const [errorStatus, setErrorStatus] = useState(null);
  const [retryCount, setRetryCount] = useState(0);

  const fetchData = async () => {
    setLoading(true);
    try {
      // 1. Check if the IP is still the placeholder
      if (API_BASE_URL.includes("http://85.215.220.246:11443/")) {
        throw new Error("PLACEHOLDER_IP");
      }

      const controller = new AbortController();
      const timeoutId = setTimeout(() => controller.abort(), 8000);

      // Fetch Stats
      const statsRes = await fetch(`${API_BASE_URL}/api/stats`, { 
        signal: controller.signal,
        mode: 'cors' 
      }).catch(err => {
        if (err.name === 'AbortError') throw new Error("TIMEOUT");
        throw err;
      });

      if (!statsRes.ok) throw new Error(`HTTP_${statsRes.status}`);
      const statsData = await statsRes.json();
      setStats(statsData);

      // Fetch Recent
      const recentRes = await fetch(`${API_BASE_URL}/api/recent`, { 
        signal: controller.signal,
        mode: 'cors'
      });
      if (!recentRes.ok) throw new Error(`HTTP_${recentRes.status}`);
      const recentData = await recentRes.json();
      setRecentStrikes(recentData);
      
      setErrorStatus(null);
      clearTimeout(timeoutId);
    } catch (error) {
      console.error("R.O.S.E. Connection Error:", error);
      
      if (error.message === "PLACEHOLDER_IP") {
        setErrorStatus("CONFIG_REQUIRED: Update YOUR_WHIPSBYTE_IP in the code.");
      } else if (error.message === "TIMEOUT") {
        setErrorStatus("TIMEOUT: VPS is not responding in time.");
      } else if (error.message.includes("Failed to fetch")) {
        setErrorStatus("NETWORK_ERROR: Browser blocked the request or VPS is unreachable.");
      } else {
        setErrorStatus(`OFFLINE: ${error.message}`);
      }
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData();
    const interval = setInterval(fetchData, 15000); // Auto-refresh every 15s
    return () => clearInterval(interval);
  }, []);

  return (
    <div className="min-h-screen bg-black text-rose-500 font-mono p-4 lg:p-8 selection:bg-rose-900 selection:text-white">
      {/* Background Decor */}
      <div className="fixed inset-0 pointer-events-none opacity-20 overflow-hidden">
        <div className="absolute top-0 left-0 w-full h-full bg-[linear-gradient(rgba(18,18,18,0)_50%,rgba(0,0,0,0.25)_50%),linear-gradient(90deg,rgba(255,0,0,0.06),rgba(0,255,0,0.02),rgba(0,0,255,0.06))] bg-[length:100%_4px,3px_100%]" />
      </div>

      {/* Header */}
      <header className="relative z-10 flex flex-col md:flex-row justify-between items-center mb-8 border-b border-rose-900/50 pb-4 gap-4">
        <div className="flex items-center gap-3">
          <div className="bg-rose-900/20 p-2 rounded border border-rose-500/50 shadow-[0_0_20px_rgba(225,29,72,0.2)]">
            <Shield className="w-8 h-8" />
          </div>
          <div>
            <h1 className="text-2xl font-black tracking-tighter text-white uppercase italic">R.O.S.E. COMMAND</h1>
            <p className="text-[10px] text-rose-400/60 uppercase tracking-[0.2em]">Tactical Intelligence Dashboard</p>
          </div>
        </div>
        
        <div className="flex items-center gap-4">
          <button 
            onClick={() => fetchData()}
            className="p-2 hover:bg-rose-900/20 rounded border border-rose-900 transition-colors"
            title="Manual Re-sync"
          >
            <RefreshCw className={`w-4 h-4 ${loading ? 'animate-spin' : ''}`} />
          </button>
          <div className="flex gap-4 text-[10px] bg-rose-950/20 px-4 py-2 rounded border border-rose-900">
            <div className="flex items-center gap-2">
              <div className={`w-2 h-2 rounded-full ${errorStatus ? 'bg-red-500 shadow-[0_0_8px_#ef4444]' : loading ? 'bg-yellow-500' : 'bg-green-500 animate-pulse shadow-[0_0_8px_#22c55e]'}`} />
              <span className={errorStatus ? 'text-red-400 font-bold' : 'text-rose-300'}>
                {errorStatus ? 'CONNECTION_LOST' : (loading ? 'SYNCING...' : 'ENCRYPTED_UPLINK')}
              </span>
            </div>
          </div>
        </div>
      </header>

      {/* Connection Diagnostic Helper */}
      {errorStatus && (
        <div className="relative z-10 mb-8 p-6 border-2 border-red-900/50 bg-red-950/10 rounded backdrop-blur-sm shadow-xl">
          <div className="flex items-start gap-4">
            <WifiOff className="w-8 h-8 text-red-500 shrink-0 mt-1" />
            <div className="space-y-3">
              <h3 className="text-lg font-bold text-red-400 uppercase tracking-tight">Critical Uplink Failure</h3>
              <p className="text-sm text-red-200/70">The dashboard cannot reach the backend server at <span className="font-bold text-white underline">{API_BASE_URL}</span>.</p>
              
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4 text-[11px] uppercase tracking-wider">
                <div className="p-3 bg-red-900/20 rounded border border-red-800">
                  <span className="block text-red-400 font-bold mb-1">Check 1: IP Address</span>
                  Current IP in code is placeholder or incorrect. Ensure it matches your Whipsbyte IP.
                </div>
                <div className="p-3 bg-red-900/20 rounded border border-red-800">
                  <span className="block text-red-400 font-bold mb-1">Check 2: api.py Status</span>
                  Run `ps aux | grep api.py` on your VPS to confirm the script is active.
                </div>
                <div className="p-3 bg-red-900/20 rounded border border-red-800">
                  <span className="block text-red-400 font-bold mb-1">Check 3: Ports & Firewall</span>
                  Ensure port 5000 is open. Check Whipsbyte firewall or run `ufw allow 5000`.
                </div>
                <div className="p-3 bg-red-900/20 rounded border border-red-800">
                  <span className="block text-red-400 font-bold mb-1">Check 4: Mixed Content</span>
                  If this site is https://, browsers block http:// API calls. Use http:// for the site too.
                </div>
              </div>
            </div>
          </div>
        </div>
      )}

      {/* Main Stats */}
      <div className="relative z-10 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-8">
        {[
          { label: 'Intelligence Records', value: stats.totalScanned.toLocaleString(), icon: Users },
          { label: 'Confirmed Threats', value: stats.threatsDetected.toLocaleString(), icon: AlertTriangle, color: 'text-rose-400 shadow-[0_0_10px_rgba(225,29,72,0.1)]' },
          { label: 'Node Status', value: errorStatus ? 'OFFLINE' : 'ACTIVE', icon: Activity, color: errorStatus ? 'text-red-500' : 'text-green-400' },
          { label: 'Kernel Threads', value: errorStatus ? '0' : '10', icon: Terminal },
        ].map((item, idx) => (
          <div key={idx} className={`bg-rose-950/10 border ${errorStatus ? 'border-red-900/30' : 'border-rose-900'} p-6 rounded transition-all hover:bg-rose-900/5`}>
            <div className="flex justify-between items-start mb-2">
              <span className="text-[10px] uppercase text-rose-400/50 font-black tracking-widest">{item.label}</span>
              <item.icon className={`w-4 h-4 ${errorStatus ? 'text-red-900' : 'text-rose-500'}`} />
            </div>
            <div className={`text-3xl font-black ${item.color || 'text-white'}`}>{item.value}</div>
          </div>
        ))}
      </div>

      {/* Log Feed */}
      <div className="relative z-10 bg-rose-950/10 border border-rose-900/50 rounded overflow-hidden shadow-2xl backdrop-blur-sm">
        <div className="p-4 border-b border-rose-900/50 bg-rose-900/10 text-xs font-black uppercase tracking-[0.3em] flex justify-between items-center">
          <div className="flex items-center gap-2">
            <Search className="w-3 h-3" />
            <span>Target Acquisition Stream</span>
          </div>
          {loading && <div className="text-[8px] animate-pulse text-rose-400">DATA_RECOVERY_IN_PROGRESS...</div>}
        </div>
        <div className="overflow-x-auto">
          <table className="w-full text-left text-sm">
            <thead className="text-rose-400/40 uppercase text-[9px] border-b border-rose-900/30 font-bold tracking-widest">
              <tr>
                <th className="p-5">Subject Identifier</th>
                <th className="p-5">Violation Classification</th>
                <th className="p-5">System Timestamp</th>
              </tr>
            </thead>
            <tbody className="divide-y divide-rose-900/10">
              {recentStrikes.map((strike, i) => (
                <tr key={i} className="hover:bg-rose-900/5 transition-colors group">
                  <td className="p-5">
                    <div className="flex items-center gap-4">
                      <div className="relative">
                        <img 
                          src={`https://www.roblox.com/headshot-thumbnail/image?userId=${strike.id}&width=48&height=48&format=png`} 
                          onError={(e) => { e.target.src = "https://tr.rbxcdn.com/38c6edcf0660a973030ca88c39a9040c/420/420/AvatarHeadshot/Png"; }}
                          className="w-10 h-10 rounded border border-rose-900 group-hover:border-rose-500 transition-colors" 
                          alt="target" 
                        />
                        <div className="absolute inset-0 bg-rose-500/10 mix-blend-overlay" />
                      </div>
                      <span className="text-white font-black tracking-tight uppercase group-hover:text-rose-400">{strike.user}</span>
                    </div>
                  </td>
                  <td className="p-5">
                    <span className="px-3 py-1 rounded text-[9px] font-black bg-rose-900/20 border border-rose-800 text-rose-400 uppercase tracking-tighter">
                      {strike.reason}
                    </span>
                  </td>
                  <td className="p-5 text-rose-400/40 font-mono text-[10px]">{strike.time}</td>
                </tr>
              ))}
              {recentStrikes.length === 0 && !errorStatus && (
                <tr>
                  <td colSpan="3" className="p-20 text-center text-rose-900 font-bold uppercase tracking-[0.5em] text-xs">
                    No Active Signals Detected
                  </td>
                </tr>
              )}
            </tbody>
          </table>
        </div>
      </div>

      {/* Footer Branding */}
      <footer className="mt-8 text-center">
        <p className="text-[9px] text-rose-900 font-black tracking-[1em] uppercase">Sector_04 Autonomous Network</p>
      </footer>
    </div>
  );
};

export default App;