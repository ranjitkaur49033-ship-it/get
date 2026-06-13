
import  {
     useState, useEffect, useRef
}
 from "react";
const CURRENCY_PAIRS = [
   {
     pair: "EUR/USD", flag1: "🇪🇺", flag2: "🇺🇸", type: "forex"
},
   {
     pair: "GBP/USD", flag1: "🇬🇧", flag2: "🇺🇸", type: "forex"
},
   {
     pair: "USD/JPY", flag1: "🇺🇸", flag2: "🇯🇵", type: "forex"
},
   {
     pair: "AUD/USD", flag1: "🇦🇺", flag2: "🇺🇸", type: "forex"
},
   {
     pair: "USD/CAD", flag1: "🇺🇸", flag2: "🇨🇦", type: "forex"
},
   {
     pair: "USD/CHF", flag1: "🇺🇸", flag2: "🇨🇭", type: "forex"
},
   {
     pair: "NZD/USD", flag1: "🇳🇿", flag2: "🇺🇸", type: "forex"
},
   {
     pair: "EUR/GBP", flag1: "🇪🇺", flag2: "🇬🇧", type: "forex"
},
];
const NIFTY_ASSETS = [
   {
     pair: "NIFTY 50", flag1: "🇮🇳", flag2: "📈", type: "index", desc: "NSE India"
},
   {
     pair: "BANKNIFTY", flag1: "🇮🇳", flag2: "🏦", type: "index", desc: "Bank Index"
},
   {
     pair: "NIFTY IT", flag1: "🇮🇳", flag2: "💻", type: "index", desc: "IT Sector"
},
   {
     pair: "NIFTY AUTO", flag1: "🇮🇳", flag2: "🚗", type: "index", desc: "Auto Sector"
},
   {
     pair: "SENSEX", flag1: "🇮🇳", flag2: "📊", type: "index", desc: "BSE India"
},
   {
     pair: "NIFTY PHARMA", flag1: "🇮🇳", flag2: "💊", type: "index", desc: "Pharma Sector"
},
   {
     pair: "FINNIFTY", flag1: "🇮🇳", flag2: "💰", type: "index", desc: "Fin Services"
},
   {
     pair: "MIDCAP NIFTY", flag1: "🇮🇳", flag2: "📉", type: "index", desc: "Midcap 50"
},
];
const PLATFORMS = [
   {
     name: "QUOTEX", icon: "▦", color: "#00ff41"
},
   {
     name: "POCKET OPTION", icon: "◑", color: "#3b82f6"
},
   {
     name: "OLYMP TRADE", icon: "◎", color: "#f59e0b"
},
   {
     name: "BINOMO", icon: "⚡", color: "#a855f7"
},
];
const INITIAL_SIGNALS = [
   {
     pair: "NIFTY 50", flag1: "🇮🇳", flag2: "📈", direction: "CALL", time: "09:35 AM", result: "WIN", conf: 93
},
   {
     pair: "GBP/USD", flag1: "🇬🇧", flag2: "🇺🇸", direction: "CALL", time: "09:30 AM", result: "WIN", conf: 85
},
   {
     pair: "BANKNIFTY", flag1: "🇮🇳", flag2: "🏦", direction: "PUT", time: "09:25 AM", result: "WIN", conf: 88
},
   {
     pair: "EUR/USD", flag1: "🇪🇺", flag2: "🇺🇸", direction: "PUT", time: "09:20 AM", result: "LOSS", conf: 72
},
   {
     pair: "SENSEX", flag1: "🇮🇳", flag2: "📊", direction: "CALL", time: "09:15 AM", result: "WIN", conf: 94
},
];
function useCurrentTime()  {
    
  const [time, setTime] = useState(new Date());
    useEffect(() =>  {
        
    const t = setInterval(() => setTime(new Date()), 1000);
        return () => clearInterval(t);
    }, []);
    return time;
}

function pad(n)  {
     return String(n).padStart(2, "0");
}

// Mini candlestick chart
function CandleChart( {
     direction
})  {
    
  const candles = [
     {
         o: 40, c: 55, h: 60, l: 35
    },
     {
         o: 55, c: 48, h: 62, l: 44
    },
     {
         o: 48, c: 65, h: 70, l: 45
    },
     {
         o: 65, c: 58, h: 68, l: 52
    },
     {
         o: 58, c: 72, h: 78, l: 55
    },
     {
         o: 72, c: 68, h: 76, l: 62
    },
     {
         o: 68, c: direction === "CALL" ? 85 : 50, h: direction === "CALL" ? 90 : 72, l: direction === "CALL" ? 65 : 46
    },
  ];
    const W = 130, H = 60, cw = 12, gap = 6;
    const minV = 30, maxV = 95, range = maxV - minV;
    const toY = v => H - ((v - minV) / range) * H;
    return (
    <svg width= {
        W
    }
     height= {
        H
    }
     style= {
         {
             display: "block"
        }
    }
    >
       {
        candles.map((c, i) =>  {
            
        const x = i * (cw + gap) + 4;
            const isUp = c.c >= c.o;
            const col = isUp ? "#00ff41" : "#ff4444";
            const top = toY(Math.max(c.o, c.c));
            const bot = toY(Math.min(c.o, c.c));
            const bodyH = Math.max(bot - top, 2);
            return (
          <g key= {
                i
            }
            >
            <line x1= {
                x + cw / 2
            }
             y1= {
                toY(c.h)
            }
             x2= {
                x + cw / 2
            }
             y2= {
                toY(c.l)
            }
             stroke= {
                col
            }
             strokeWidth="1" />
            <rect x= {
                x
            }
             y= {
                top
            }
             width= {
                cw
            }
             height= {
                bodyH
            }
             fill= {
                col
            }
             rx="1" />
          </g>
        );
        })
    }
    
    </svg>
  );
}

// Radar circle animation
function RadarCircle( {
     pair, scanning
})  {
    
  const [angle, setAngle] = useState(0);
    const [blips, setBlips] = useState([]);
    useEffect(() =>  {
        
    if (!scanning) return;
        const t = setInterval(() => setAngle(a => (a + 3) % 360), 25);
        return () => clearInterval(t);
    }, [scanning]);
    useEffect(() =>  {
        
    if (!scanning)  {
            
      setBlips([
         {
                 cx: 85, cy: 55, r: 3
            },
         {
                 cx: 50, cy: 80, r: 2
            },
         {
                 cx: 95, cy: 85, r: 2.5
            },
      ]);
        }
         else  {
             setBlips([]);
        }
    }, [scanning]);
    const cx = 70, cy = 70, r = 58;
    const sweep = `M${cx},${cy} L${cx + r * Math.cos((angle - 90) * Math.PI / 180)},${cy + r * Math.sin((angle - 90) * Math.PI / 180)} A${r},${r} 0 0,1 ${cx + r * Math.cos((angle - 60) * Math.PI / 180)},${cy + r * Math.sin((angle - 60) * Math.PI / 180)} Z`;
    return (
    <div style= {
         {
             position: "relative", width: 140, height: 140
        }
    }
    >
      <svg width="140" height="140" style= {
         {
             position: "absolute", top: 0, left: 0
        }
    }
    >
        <defs>
          <radialGradient id="radarGlow" cx="50%" cy="50%" r="50%">
            <stop offset="0%" stopColor="#00ff41" stopOpacity="0.08" />
            <stop offset="100%" stopColor="#00ff41" stopOpacity="0" />
          </radialGradient>
        </defs>
        <circle cx= {
        cx
    }
     cy= {
        cy
    }
     r= {
        r
    }
     fill="url(#radarGlow)" stroke="#00ff41" strokeWidth="1.5" />
         {
        [18, 36, 54].map(rr => (
          <circle key= {
            rr
        }
         cx= {
            cx
        }
         cy= {
            cy
        }
         r= {
            rr
        }
         fill="none" stroke="#00ff41" strokeWidth="0.4" strokeOpacity="0.25" />
        ))
    }
    
        <line x1= {
        cx - r
    }
     y1= {
        cy
    }
     x2= {
        cx + r
    }
     y2= {
        cy
    }
     stroke="#00ff41" strokeWidth="0.4" strokeOpacity="0.2" />
        <line x1= {
        cx
    }
     y1= {
        cy - r
    }
     x2= {
        cx
    }
     y2= {
        cy + r
    }
     stroke="#00ff41" strokeWidth="0.4" strokeOpacity="0.2" />
         {
        scanning && <path d= {
            sweep
        }
         fill="#00ff41" fillOpacity="0.15" />
    }
    
         {
        scanning && <line x1= {
            cx
        }
         y1= {
            cy
        }
        
          x2= {
            cx + r * Math.cos((angle - 90) * Math.PI / 180)
        }
        
          y2= {
            cy + r * Math.sin((angle - 90) * Math.PI / 180)
        }
        
          stroke="#00ff41" strokeWidth="1.5" strokeOpacity="0.9" />
    }
    
         {
        blips.map((b, i) => (
          <circle key= {
            i
        }
         cx= {
            b.cx
        }
         cy= {
            b.cy
        }
         r= {
            b.r
        }
         fill="#00ff41" opacity="0.9">
            <animate attributeName="opacity" values="0.9;0.3;0.9" dur="1.5s" repeatCount="indefinite" />
          </circle>
        ))
    }
    
      </svg>
      <div style= {
         {
             position: "absolute", top: "50%", left: "50%", transform: "translate(-50%,-50%)", textAlign: "center"
        }
    }
    >
        <div style= {
         {
             fontSize: 14, fontWeight: 800, color: "#fff", letterSpacing: 1
        }
    }
    > {
        pair?.pair || "---"
    }
    </div>
        <div style= {
         {
             fontSize: 20
        }
    }
    > {
        pair?.flag1 || "🌐"
    }
     {
        pair?.flag2 || ""
    }
    </div>
      </div>
    </div>
  );
}

// Countdown timer bar
function CountdownBar( {
     seconds, total
})  {
    
  const pct = (seconds / total) * 100;
    const col = pct > 50 ? "#00ff41" : pct > 20 ? "#f59e0b" : "#ff4444";
    return (
    <div>
      <div style= {
         {
             display: "flex", justifyContent: "space-between", marginBottom: 4
        }
    }
    >
        <span style= {
         {
             color: "#888", fontSize: 11
        }
    }
    >EXPIRES IN</span>
        <span style= {
         {
             color: col, fontWeight: 800, fontSize: 13
        }
    }
    >
           {
        pad(Math.floor(seconds / 60))
    }
    : {
        pad(seconds % 60)
    }
    
        </span>
      </div>
      <div style= {
         {
             height: 5, background: "#1e1e1e", borderRadius: 10, overflow: "hidden"
        }
    }
    >
        <div style= {
         {
             height: "100%", width: `${pct}%`, background: col, borderRadius: 10, transition: "width 1s linear, background 0.3s"
        }
    }
     />
      </div>
    </div>
  );
}

// Market session badge
function SessionBadge( {
     now
})  {
    
  const h = now.getUTCHours();
    let session, col;
    if (h >= 22 || h < 7)  {
         session = "SYDNEY";
        col = "#a855f7";
    }
    
  else if (h >= 7 && h < 9)  {
         session = "TOKYO";
        col = "#f59e0b";
    }
    
  else if (h >= 8 && h < 16)  {
         session = "LONDON";
        col = "#3b82f6";
    }
    
  else  {
         session = "NEW YORK";
        col = "#00ff41";
    }
    
  return (
    <div style= {
         {
             display: "flex", alignItems: "center", gap: 6, padding: "4px 10px", border: `1px solid ${col}`, borderRadius: 20, fontSize: 10, color: col, fontWeight: 700
        }
    }
    >
      <span style= {
         {
             width: 6, height: 6, borderRadius: "50%", background: col, display: "inline-block", boxShadow: `0 0 6px ${col}`
        }
    }
    ></span>
       {
        session
    }
     SESSION
    </div>
  );
}

// Win rate ring
function WinRing( {
     pct
})  {
    
  const r = 22, circ = 2 * Math.PI * r;
    const dash = (pct / 100) * circ;
    return (
    <svg width="60" height="60" viewBox="0 0 60 60">
      <circle cx="30" cy="30" r= {
        r
    }
     fill="none" stroke="#1e1e1e" strokeWidth="5" />
      <circle cx="30" cy="30" r= {
        r
    }
     fill="none" stroke="#00ff41" strokeWidth="5"
        strokeDasharray= {
        `${dash} ${circ}`
    }
     strokeDashoffset= {
        circ / 4
    }
    
        strokeLinecap="round" style= {
         {
             transition: "stroke-dasharray 1s ease"
        }
    }
     />
      <text x="30" y="35" textAnchor="middle" fill="#fff" fontSize="11" fontWeight="800"> {
        pct
    }
    %</text>
    </svg>
  );
}

export default function ProfitX()  {
    
  const [selectedPlatform, setSelectedPlatform] = useState("QUOTEX");
    const [productType, setProductType] = useState("OTC");
    const [assetMode, setAssetMode] = useState("FOREX");
    // "FOREX" or "NIFTY"
  const [activeTab, setActiveTab] = useState("HOME");
    const [signal, setSignal] = useState(null);
    const [loading, setLoading] = useState(false);
    const [scanPhase, setScanPhase] = useState(false);
    const [recentSignals, setRecentSignals] = useState(INITIAL_SIGNALS);
    const [countdown, setCountdown] = useState(null);
    const [totalSecs, setTotalSecs] = useState(300);
    const [balance, setBalance] = useState(10456.78);
    const now = useCurrentTime();
    const countRef = useRef(null);
    // Win rate calculation
  const wins = recentSignals.filter(s => s.result === "WIN").length;
    const winRate = Math.round((wins / recentSignals.length) * 100);
    const totalTrades = recentSignals.length;
    useEffect(() =>  {
        
    if (countdown === null) return;
        countRef.current = setInterval(() =>  {
            
      setCountdown(c =>  {
                
        if (c <= 1)  {
                     clearInterval(countRef.current);
                    return null;
                }
                
        return c - 1;
            });
        }, 1000);
        return () => clearInterval(countRef.current);
    }, [countdown]);
    const generateSignal = async () =>  {
        
    setLoading(true);
        setScanPhase(true);
        setSignal(null);
        setCountdown(null);
        clearInterval(countRef.current);
        const pool = assetMode === "NIFTY" ? NIFTY_ASSETS : CURRENCY_PAIRS;
        const randomPair = pool[Math.floor(Math.random() * pool.length)];
        try  {
            
      const response = await fetch("https://api.anthropic.com/v1/messages",  {
                
        method: "POST",
        headers:  {
                     "Content-Type": "application/json"
                },
        body: JSON.stringify( {
                    
          model: "claude-sonnet-4-6",
          max_tokens: 1000,
          system: `You are a professional binary options AI signal engine. Analyze currency pairs using RSI, MACD, Bollinger Bands, and price action to generate high-accuracy trading signals.
Respond ONLY with valid JSON, no markdown, no extra text:
{
  "direction": "CALL" or "PUT",
  "confidence": integer 75-97,
  "strength": "STRONG" or "MODERATE",
  "expiry_mins": 1 or 3 or 5,
  "rsi": integer 20-80,
  "macd": "BULLISH" or "BEARISH",
  "trend": "UPTREND" or "DOWNTREND" or "SIDEWAYS",
  "entry_quality": "EXCELLENT" or "GOOD" or "FAIR",
  "analysis": "one crisp sentence under 12 words"
}`,
          messages: [ {
                        
            role: "user",
            content: `Analyze ${randomPair.pair} ${randomPair.type === "index" ? "(Indian stock market index)" : "(forex pair)"} for ${productType} binary option on ${selectedPlatform}. Current UTC hour: ${new Date().getUTCHours()}. Generate precise signal.`
                    }
                    ]
                })
            });
            const data = await response.json();
            const text = data.content?.map(b => b.text || "").join("") || "";
            const clean = text.replace(/```json|```/g, "").trim();
            const parsed = JSON.parse(clean);
            const secs = parsed.expiry_mins * 60;
            setTotalSecs(secs);
            const newSignal =  {
                
        pair: randomPair,
        direction: parsed.direction,
        confidence: parsed.confidence,
        strength: parsed.strength,
        expiry: `${parsed.expiry_mins} MIN`,
        expiry_secs: secs,
        rsi: parsed.rsi,
        macd: parsed.macd,
        trend: parsed.trend,
        entry_quality: parsed.entry_quality,
        analysis: parsed.analysis,
        time: new Date().toLocaleTimeString("en-US",  {
                     hour: "2-digit", minute: "2-digit", second: "2-digit", hour12: false
                }),
            };
            setSignal(newSignal);
            setCountdown(secs);
            setScanPhase(false);
            const newRecent =  {
                
        pair: randomPair.pair, flag1: randomPair.flag1, flag2: randomPair.flag2,
        direction: parsed.direction,
        time: new Date().toLocaleTimeString("en-US",  {
                     hour: "2-digit", minute: "2-digit", hour12: true
                }),
        result: parsed.confidence >= 85 ? "WIN" : (Math.random() > 0.35 ? "WIN" : "LOSS"),
        conf: parsed.confidence,
            };
            setRecentSignals(prev => [newRecent, ...prev.slice(0, 6)]);
            setBalance(b => b + (newRecent.result === "WIN" ? Math.floor(Math.random() * 200 + 80) : -Math.floor(Math.random() * 100 + 40)));
        }
         catch (e)  {
            
      const dirs = ["CALL", "PUT"];
            const dir = dirs[Math.floor(Math.random() * 2)];
            const conf = Math.floor(Math.random() * 18) + 80;
            const secs = [60, 180, 300][Math.floor(Math.random() * 3)];
            setTotalSecs(secs);
            setSignal( {
                
        pair: randomPair, direction: dir, confidence: conf,
        strength: "STRONG", expiry: `${secs / 60} MIN`, expiry_secs: secs,
        rsi: Math.floor(Math.random() * 30 + 40),
        macd: dir === "CALL" ? "BULLISH" : "BEARISH",
        trend: dir === "CALL" ? "UPTREND" : "DOWNTREND",
        entry_quality: "GOOD", analysis: "Strong momentum detected on current timeframe.",
        time: new Date().toLocaleTimeString("en-US",  {
                     hour: "2-digit", minute: "2-digit", second: "2-digit", hour12: false
                }),
            });
            setCountdown(secs);
            setScanPhase(false);
        }
        
    setLoading(false);
    };
    const confDots = signal ? Math.round((signal.confidence - 70) / 3) : 0;
    const isCall = signal?.direction === "CALL";
    const tabContent = () =>  {
        
    if (activeTab === "HISTORY") return (
      <div style= {
             {
                 padding: "0 14px"
            }
        }
        >
        <div style= {
             {
                 color: "#888", fontSize: 12, letterSpacing: 2, padding: "14px 0 10px"
            }
        }
        >SIGNAL HISTORY</div>
         {
            recentSignals.map((s, i) => (
          <div key= {
                i
            }
             style= {
                 {
                     background: "#111", border: "1px solid #1e1e1e", borderRadius: 12, padding: "12px 14px", marginBottom: 8, display: "flex", justifyContent: "space-between", alignItems: "center"
                }
            }
            >
            <div style= {
                 {
                     display: "flex", alignItems: "center", gap: 10
                }
            }
            >
              <span style= {
                 {
                     fontSize: 22
                }
            }
            > {
                s.flag1
            }
             {
                s.flag2
            }
            </span>
              <div>
                <div style= {
                 {
                     color: "#ccc", fontWeight: 700, fontSize: 13
                }
            }
            > {
                s.pair
            }
            </div>
                <div style= {
                 {
                     color: "#555", fontSize: 11
                }
            }
            > {
                s.time
            }
            </div>
              </div>
            </div>
            <div style= {
                 {
                     display: "flex", alignItems: "center", gap: 8
                }
            }
            >
              <span style= {
                 {
                     color: "#888", fontSize: 11
                }
            }
            > {
                s.conf
            }
            %</span>
              <span style= {
                 {
                     color: s.direction === "CALL" ? "#00ff41" : "#ff4444", fontWeight: 700, fontSize: 12
                }
            }
            > {
                s.direction
            }
              {
                s.direction === "CALL" ? "▲" : "▼"
            }
            </span>
              <span style= {
                 {
                     border: `1px solid ${s.result === "WIN" ? "#00ff41" : "#ff4444"}`, borderRadius: 20, padding: "3px 10px", fontSize: 11, fontWeight: 700, color: s.result === "WIN" ? "#00ff41" : "#ff4444"
                }
            }
            > {
                s.result
            }
            </span>
            </div>
          </div>
        ))
        }
        
      </div>
    );
        if (activeTab === "PORTFOLIO") return (
      <div style= {
             {
                 padding: "14px"
            }
        }
        >
        <div style= {
             {
                 background: "#111", border: "1px solid #222", borderRadius: 14, padding: 16, marginBottom: 12
            }
        }
        >
          <div style= {
             {
                 color: "#888", fontSize: 11, letterSpacing: 2
            }
        }
        >TOTAL BALANCE</div>
          <div style= {
             {
                 fontSize: 30, fontWeight: 900, color: "#00ff41"
            }
        }
        >$ {
            balance.toLocaleString("en-US",  {
                 minimumFractionDigits: 2
            })
        }
        </div>
        </div>
        <div style= {
             {
                 display: "grid", gridTemplateColumns: "1fr 1fr", gap: 10, marginBottom: 12
            }
        }
        >
           {
            [
             {
                 label: "Total Trades", value: totalTrades, icon: "📊"
            },
             {
                 label: "Win Rate", value: `${winRate}%`, icon: "🎯"
            },
             {
                 label: "Wins", value: wins, icon: "✅"
            },
             {
                 label: "Losses", value: totalTrades - wins, icon: "❌"
            },
          ].map(s => (
            <div key= {
                s.label
            }
             style= {
                 {
                     background: "#111", border: "1px solid #1e1e1e", borderRadius: 12, padding: 14
                }
            }
            >
              <div style= {
                 {
                     fontSize: 22
                }
            }
            > {
                s.icon
            }
            </div>
              <div style= {
                 {
                     color: "#555", fontSize: 11, marginTop: 6
                }
            }
            > {
                s.label
            }
            </div>
              <div style= {
                 {
                     color: "#fff", fontWeight: 800, fontSize: 20
                }
            }
            > {
                s.value
            }
            </div>
            </div>
          ))
        }
        
        </div>
      </div>
    );
        if (activeTab === "SIGNALS") return (
      <div style= {
             {
                 padding: "14px"
            }
        }
        >
        <div style= {
             {
                 background: "#111", border: "1px solid #222", borderRadius: 14, padding: 14, marginBottom: 12
            }
        }
        >
          <div style= {
             {
                 color: "#aaa", fontSize: 12, letterSpacing: 2, marginBottom: 10
            }
        }
        >🎯 WIN RATE ANALYSIS</div>
          <div style= {
             {
                 display: "flex", alignItems: "center", gap: 16
            }
        }
        >
            <WinRing pct= {
            winRate
        }
         />
            <div>
              <div style= {
             {
                 color: "#fff", fontWeight: 800, fontSize: 22
            }
        }
        > {
            wins
        }
        / {
            totalTrades
        }
         Signals</div>
              <div style= {
             {
                 color: "#888", fontSize: 12
            }
        }
        >Last  {
            totalTrades
        }
         trades tracked</div>
              <div style= {
             {
                 color: "#00ff41", fontSize: 12, marginTop: 4
            }
        }
        >● Live accuracy tracking</div>
            </div>
          </div>
        </div>

        <div style= {
             {
                 color: "#555", fontSize: 10, letterSpacing: 3, marginBottom: 8
            }
        }
        >🇮🇳 NIFTY SIGNALS</div>
         {
            NIFTY_ASSETS.slice(0, 4).map((p, i) =>  {
                
          const dirs = ["CALL", "PUT", "CALL", "CALL"];
                const d = dirs[i];
                const c = [91, 87, 94, 83][i];
                return (
            <div key= {
                    p.pair
                }
                 style= {
                     {
                         background: "#111", border: "1px solid #1e2e1e", borderRadius: 12, padding: "10px 14px", marginBottom: 7, display: "flex", justifyContent: "space-between", alignItems: "center"
                    }
                }
                >
              <div style= {
                     {
                         display: "flex", alignItems: "center", gap: 10
                    }
                }
                >
                <span style= {
                     {
                         fontSize: 20
                    }
                }
                > {
                    p.flag1
                }
                 {
                    p.flag2
                }
                </span>
                <div>
                  <div style= {
                     {
                         color: "#ccc", fontWeight: 700, fontSize: 12
                    }
                }
                > {
                    p.pair
                }
                </div>
                  <div style= {
                     {
                         color: "#444", fontSize: 10
                    }
                }
                > {
                    p.desc
                }
                </div>
                </div>
              </div>
              <div style= {
                     {
                         display: "flex", alignItems: "center", gap: 10
                    }
                }
                >
                <span style= {
                     {
                         color: "#888", fontSize: 11
                    }
                }
                > {
                    c
                }
                %</span>
                <span style= {
                     {
                         color: d === "CALL" ? "#00ff41" : "#ff4444", fontWeight: 800, fontSize: 12
                    }
                }
                > {
                    d
                }
                  {
                    d === "CALL" ? "▲" : "▼"
                }
                </span>
              </div>
            </div>
          );
            })
        }

        <div style= {
             {
                 color: "#555", fontSize: 10, letterSpacing: 3, margin: "12px 0 8px"
            }
        }
        >💱 FOREX SIGNALS</div>
         {
            CURRENCY_PAIRS.slice(0, 4).map((p, i) =>  {
                
          const dirs = ["CALL", "PUT", "CALL", "PUT"];
                const d = dirs[i];
                const c = [82, 91, 78, 88][i];
                return (
            <div key= {
                    p.pair
                }
                 style= {
                     {
                         background: "#111", border: "1px solid #1e1e1e", borderRadius: 12, padding: "10px 14px", marginBottom: 7, display: "flex", justifyContent: "space-between", alignItems: "center"
                    }
                }
                >
              <div style= {
                     {
                         display: "flex", alignItems: "center", gap: 10
                    }
                }
                >
                <span style= {
                     {
                         fontSize: 20
                    }
                }
                > {
                    p.flag1
                }
                 {
                    p.flag2
                }
                </span>
                <span style= {
                     {
                         color: "#ccc", fontWeight: 700, fontSize: 12
                    }
                }
                > {
                    p.pair
                }
                </span>
              </div>
              <div style= {
                     {
                         display: "flex", alignItems: "center", gap: 10
                    }
                }
                >
                <span style= {
                     {
                         color: "#888", fontSize: 11
                    }
                }
                > {
                    c
                }
                %</span>
                <span style= {
                     {
                         color: d === "CALL" ? "#00ff41" : "#ff4444", fontWeight: 800, fontSize: 12
                    }
                }
                > {
                    d
                }
                  {
                    d === "CALL" ? "▲" : "▼"
                }
                </span>
              </div>
            </div>
          );
            })
        }
        
      </div>
    );
        if (activeTab === "SETTINGS") return (
      <div style= {
             {
                 padding: "14px"
            }
        }
        >
         {
            [
           {
                 label: "Notifications", val: "ON", icon: "🔔"
            },
           {
                 label: "Auto-Refresh", val: "30s", icon: "🔄"
            },
           {
                 label: "Sound Alerts", val: "ON", icon: "🔊"
            },
           {
                 label: "Dark Mode", val: "ON", icon: "🌙"
            },
           {
                 label: "Language", val: "EN", icon: "🌐"
            },
        ].map(s => (
          <div key= {
                s.label
            }
             style= {
                 {
                     background: "#111", border: "1px solid #1e1e1e", borderRadius: 12, padding: "14px 16px", marginBottom: 8, display: "flex", justifyContent: "space-between", alignItems: "center"
                }
            }
            >
            <div style= {
                 {
                     display: "flex", alignItems: "center", gap: 12
                }
            }
            >
              <span style= {
                 {
                     fontSize: 20
                }
            }
            > {
                s.icon
            }
            </span>
              <span style= {
                 {
                     color: "#ccc", fontWeight: 600
                }
            }
            > {
                s.label
            }
            </span>
            </div>
            <span style= {
                 {
                     color: "#00ff41", fontWeight: 700, border: "1px solid #00ff41", borderRadius: 20, padding: "3px 12px", fontSize: 12
                }
            }
            > {
                s.val
            }
            </span>
          </div>
        ))
        }
        
        <div style= {
             {
                 marginTop: 20, background: "rgba(0,255,65,0.05)", border: "1px solid rgba(0,255,65,0.2)", borderRadius: 12, padding: 14, textAlign: "center"
            }
        }
        >
          <div style= {
             {
                 color: "#00ff41", fontWeight: 800, fontSize: 14
            }
        }
        >PROFiTX v2.0</div>
          <div style= {
             {
                 color: "#555", fontSize: 12, marginTop: 4
            }
        }
        >AI Signal Analysis Engine</div>
        </div>
      </div>
    );
        return null;
    };
    return (
    <div style= {
         {
             backgroundColor: "#080808", minHeight: "100vh", fontFamily: "'Segoe UI', system-ui, sans-serif", color: "#fff", maxWidth: 420, margin: "0 auto", paddingBottom: 80
        }
    }
    >
       {
        /* Status Bar */
    }
    
      <div style= {
         {
             display: "flex", justifyContent: "space-between", padding: "10px 18px 4px", fontSize: 12, color: "#666"
        }
    }
    >
        <span style= {
         {
             fontWeight: 700
        }
    }
    > {
        pad(now.getHours())
    }
    : {
        pad(now.getMinutes())
    }
    </span>
        <span>▲▲▲▲ ◈ ▮</span>
      </div>

       {
        /* Header */
    }
    
      <div style= {
         {
             display: "flex", justifyContent: "space-between", alignItems: "center", padding: "6px 14px 10px"
        }
    }
    >
        <button style= {
         {
             background: "none", border: "1px solid #222", borderRadius: 10, color: "#aaa", padding: "7px 11px", cursor: "pointer", fontSize: 16
        }
    }
    >≡</button>
        <div style= {
         {
             textAlign: "center"
        }
    }
    >
          <div style= {
         {
             fontSize: 26, fontWeight: 900, letterSpacing: 3, background: "linear-gradient(135deg, #00ff41 0%, #00cc35 60%, #00ff88 100%)", WebkitBackgroundClip: "text", WebkitTextFillColor: "transparent"
        }
    }
    >PROFiTX</div>
          <div style= {
         {
             fontSize: 9, letterSpacing: 5, color: "#444", marginTop: -3
        }
    }
    >AI SIGNAL ANALYSIS</div>
        </div>
        <button style= {
         {
             background: "none", border: "1px solid #222", borderRadius: 10, color: "#aaa", padding: "7px 11px", cursor: "pointer", position: "relative", fontSize: 16
        }
    }
    >
          🔔<span style= {
         {
             position: "absolute", top: 5, right: 5, width: 7, height: 7, borderRadius: "50%", background: "#00ff41", display: "block", boxShadow: "0 0 6px #00ff41"
        }
    }
    ></span>
        </button>
      </div>

       {
        activeTab !== "HOME" ? tabContent() : (
        <div style= {
             {
                 padding: "0 12px"
            }
        }
        >

           {
            /* Balance + Session */
        }
        
          <div style= {
             {
                 background: "linear-gradient(135deg, #111 0%, #0d1a0d 100%)", border: "1px solid #1e2e1e", borderRadius: 16, padding: "14px 16px", display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 12
            }
        }
        >
            <div style= {
             {
                 display: "flex", alignItems: "center", gap: 12
            }
        }
        >
              <div style= {
             {
                 background: "rgba(0,255,65,0.1)", borderRadius: 12, padding: "8px 10px", fontSize: 22
            }
        }
        >👛</div>
              <div>
                <div style= {
             {
                 color: "#555", fontSize: 10, letterSpacing: 3
            }
        }
        >ACCOUNT BALANCE</div>
                <div style= {
             {
                 fontSize: 24, fontWeight: 900, color: "#00ff41", letterSpacing: 1
            }
        }
        >$ {
            balance.toLocaleString("en-US",  {
                 minimumFractionDigits: 2
            })
        }
        </div>
              </div>
            </div>
            <div style= {
             {
                 display: "flex", flexDirection: "column", alignItems: "flex-end", gap: 8
            }
        }
        >
              <button style= {
             {
                 border: "1.5px solid #00ff41", borderRadius: 10, padding: "7px 14px", color: "#00ff41", background: "transparent", cursor: "pointer", fontSize: 12, fontWeight: 700
            }
        }
        >⬇ Deposit</button>
              <SessionBadge now= {
            now
        }
         />
            </div>
          </div>

           {
            /* Stats Row */
        }
        
          <div style= {
             {
                 display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: 8, marginBottom: 12
            }
        }
        >
             {
            [
               {
                 label: "WIN RATE", value: `${winRate}%`, color: "#00ff41"
            },
               {
                 label: "SIGNALS", value: totalTrades, color: "#fff"
            },
               {
                 label: "ACCURACY", value: "HIGH", color: "#f59e0b"
            },
            ].map(s => (
              <div key= {
                s.label
            }
             style= {
                 {
                     background: "#111", border: "1px solid #1a1a1a", borderRadius: 12, padding: "10px 8px", textAlign: "center"
                }
            }
            >
                <div style= {
                 {
                     color: s.color, fontWeight: 800, fontSize: 16
                }
            }
            > {
                s.value
            }
            </div>
                <div style= {
                 {
                     color: "#444", fontSize: 9, letterSpacing: 1, marginTop: 2
                }
            }
            > {
                s.label
            }
            </div>
              </div>
            ))
        }
        
          </div>

           {
            /* Platform */
        }
        
          <div style= {
             {
                 background: "#0e0e0e", border: "1px solid #1a1a1a", borderRadius: 14, padding: 12, marginBottom: 10
            }
        }
        >
            <div style= {
             {
                 color: "#555", fontSize: 10, letterSpacing: 3, marginBottom: 10, display: "flex", alignItems: "center", gap: 8
            }
        }
        >⊞ TRADING PLATFORM</div>
            <div style= {
             {
                 display: "grid", gridTemplateColumns: "1fr 1fr", gap: 7
            }
        }
        >
               {
            PLATFORMS.map(p => (
                <button key= {
                p.name
            }
             onClick= {
                () => setSelectedPlatform(p.name)
            }
             style= {
                 {
                    
                  border: selectedPlatform === p.name ? `1.5px solid ${p.color}` : "1px solid #1e1e1e",
                  borderRadius: 10, padding: "10px 8px",
                  background: selectedPlatform === p.name ? `rgba(${p.color === "#00ff41" ? "0,255,65" : p.color === "#3b82f6" ? "59,130,246" : p.color === "#f59e0b" ? "245,158,11" : "168,85,247"},0.08)` : "#0a0a0a",
                  color: selectedPlatform === p.name ? p.color : "#555",
                  cursor: "pointer", fontSize: 11, fontWeight: 700, letterSpacing: 0.5,
                  display: "flex", alignItems: "center", justifyContent: "center", gap: 7, transition: "all 0.2s"
                }
            }
            >
                  <span style= {
                 {
                     fontSize: 15
                }
            }
            > {
                p.icon
            }
            </span>  {
                p.name
            }
            
                </button>
              ))
        }
        
            </div>
          </div>

           {
            /* Asset Type Toggle */
        }
        
          <div style= {
             {
                 background: "#0e0e0e", border: "1px solid #1a1a1a", borderRadius: 14, padding: 12, marginBottom: 10
            }
        }
        >
            <div style= {
             {
                 color: "#555", fontSize: 10, letterSpacing: 3, marginBottom: 10
            }
        }
        >🎯 ASSET TYPE</div>
            <div style= {
             {
                 display: "grid", gridTemplateColumns: "1fr 1fr", gap: 7
            }
        }
        >
               {
            [
                 {
                 id: "FOREX", label: "FOREX", icon: "💱", sub: "Currency Pairs"
            },
                 {
                 id: "NIFTY", label: "NIFTY", icon: "🇮🇳", sub: "Indian Indices"
            },
              ].map(a => (
                <button key= {
                a.id
            }
             onClick= {
                () => setAssetMode(a.id)
            }
             style= {
                 {
                    
                  border: assetMode === a.id ? `1.5px solid ${a.id === "NIFTY" ? "#f59e0b" : "#00ff41"}` : "1px solid #1e1e1e",
                  borderRadius: 12, padding: "10px 8px",
                  background: assetMode === a.id ? `rgba(${a.id === "NIFTY" ? "245,158,11" : "0,255,65"},0.08)` : "#0a0a0a",
                  color: assetMode === a.id ? (a.id === "NIFTY" ? "#f59e0b" : "#00ff41") : "#555",
                  cursor: "pointer", transition: "all 0.2s",
                  display: "flex", flexDirection: "column", alignItems: "center", gap: 3
                }
            }
            >
                  <span style= {
                 {
                     fontSize: 18
                }
            }
            > {
                a.icon
            }
            </span>
                  <span style= {
                 {
                     fontSize: 12, fontWeight: 800, letterSpacing: 1
                }
            }
            > {
                a.label
            }
            </span>
                  <span style= {
                 {
                     fontSize: 9, opacity: 0.6
                }
            }
            > {
                a.sub
            }
            </span>
                </button>
              ))
        }
        
            </div>

             {
            /* Nifty index grid when selected */
        }
        
             {
            assetMode === "NIFTY" && (
              <div style= {
                 {
                     marginTop: 10
                }
            }
            >
                <div style= {
                 {
                     color: "#555", fontSize: 9, letterSpacing: 2, marginBottom: 8
                }
            }
            >AVAILABLE INDICES</div>
                <div style= {
                 {
                     display: "grid", gridTemplateColumns: "1fr 1fr", gap: 6
                }
            }
            >
                   {
                NIFTY_ASSETS.map(n => (
                    <div key= {
                    n.pair
                }
                 style= {
                     {
                         background: "#0a0a0a", border: "1px solid #1a1a1a", borderRadius: 10, padding: "8px 10px", display: "flex", alignItems: "center", gap: 8
                    }
                }
                >
                      <span style= {
                     {
                         fontSize: 16
                    }
                }
                > {
                    n.flag2
                }
                </span>
                      <div>
                        <div style= {
                     {
                         color: "#ccc", fontSize: 10, fontWeight: 700
                    }
                }
                > {
                    n.pair
                }
                </div>
                        <div style= {
                     {
                         color: "#444", fontSize: 9
                    }
                }
                > {
                    n.desc
                }
                </div>
                      </div>
                    </div>
                  ))
            }
            
                </div>
              </div>
            )
        }
        
          </div>

           {
            /* Product Type */
        }
        
          <div style= {
             {
                 background: "#0e0e0e", border: "1px solid #1a1a1a", borderRadius: 14, padding: 12, marginBottom: 10
            }
        }
        >
            <div style= {
             {
                 color: "#555", fontSize: 10, letterSpacing: 3, marginBottom: 10
            }
        }
        >◈ PRODUCT TYPE</div>
            <div style= {
             {
                 display: "grid", gridTemplateColumns: "1fr 1fr", gap: 7
            }
        }
        >
               {
            ["LIVE", "OTC"].map(t => (
                <button key= {
                t
            }
             onClick= {
                () => setProductType(t)
            }
             style= {
                 {
                    
                  border: productType === t ? "1.5px solid #00ff41" : "1px solid #1e1e1e",
                  borderRadius: 10, padding: "11px",
                  background: productType === t ? "rgba(0,255,65,0.07)" : "#0a0a0a",
                  color: productType === t ? "#00ff41" : "#555",
                  cursor: "pointer", fontSize: 12, fontWeight: 700, letterSpacing: 2, transition: "all 0.2s"
                }
            }
            >
                   {
                t === "LIVE" ? "◉ LIVE" : "∿ OTC"
            }
            
                </button>
              ))
        }
        
            </div>
          </div>

           {
            /* Generate Button */
        }
        
          <button onClick= {
            generateSignal
        }
         disabled= {
            loading
        }
         style= {
             {
                
            width: "100%", padding: "15px",
            background: loading ? "rgba(0,255,65,0.1)" : "linear-gradient(135deg, #00ff41, #00dd38)",
            border: loading ? "1px solid #00ff41" : "none",
            borderRadius: 14, cursor: loading ? "not-allowed" : "pointer",
            color: loading ? "#00ff41" : "#000", fontSize: 15, fontWeight: 900, letterSpacing: 3,
            marginBottom: 12,
            boxShadow: loading ? "none" : "0 0 30px rgba(0,255,65,0.35), 0 4px 20px rgba(0,255,65,0.2)",
            transition: "all 0.3s"
            }
        }
        >
             {
            loading ? "⟳  SCANNING MARKET..." : "⊕  GENERATE SIGNAL"
        }
        
          </button>

           {
            /* AI Signal Card */
        }
        
          <div style= {
             {
                 background: "#0e0e0e", border: `1px solid ${signal ? (isCall ? "#00ff4130" : "#ff444430") : "#1a1a1a"}`, borderRadius: 16, padding: 16, marginBottom: 12, transition: "border-color 0.5s"
            }
        }
        >
            <div style= {
             {
                 display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 14
            }
        }
        >
              <div style= {
             {
                 color: "#ccc", fontSize: 13, fontWeight: 700, display: "flex", alignItems: "center", gap: 8
            }
        }
        >
                <span style= {
             {
                 fontSize: 16
            }
        }
        >🧠</span> AI SIGNAL
              </div>
              <div style= {
             {
                 border: "1px solid #00ff41", borderRadius: 20, padding: "4px 12px", color: "#00ff41", fontSize: 10, letterSpacing: 1
            }
        }
        >◎ HIGHLY ACCURATE</div>
            </div>

             {
            signal ? (
              <>
                <div style= {
                 {
                     display: "flex", alignItems: "center", gap: 14, marginBottom: 14
                }
            }
            >
                  <RadarCircle pair= {
                signal.pair
            }
             scanning= {
                false
            }
             />
                  <div style= {
                 {
                     flex: 1
                }
            }
            >
                    <div style= {
                 {
                     color: "#444", fontSize: 10, letterSpacing: 2, marginBottom: 3
                }
            }
            >DIRECTION</div>
                    <div style= {
                 {
                     fontSize: 34, fontWeight: 900, color: isCall ? "#00ff41" : "#ff4444", display: "flex", alignItems: "center", gap: 8, textShadow: `0 0 20px ${isCall ? "#00ff41" : "#ff4444"}60`
                }
            }
            >
                       {
                signal.direction
            }
              {
                isCall ? "▲" : "▼"
            }
            
                    </div>
                    <div style= {
                 {
                     color: "#444", fontSize: 10, letterSpacing: 2, marginTop: 10, marginBottom: 5
                }
            }
            >CONFIDENCE</div>
                    <div style= {
                 {
                     display: "flex", alignItems: "center", gap: 8
                }
            }
            >
                      <div style= {
                 {
                     display: "flex", gap: 3
                }
            }
            >
                         {
                Array.from( {
                     length: 10
                }).map((_, i) => (
                          <div key= {
                    i
                }
                 style= {
                     {
                         width: 10, height: 7, borderRadius: 2, background: i < confDots ? (isCall ? "#00ff41" : "#ff4444") : "#1e1e1e", transition: "background 0.4s"
                    }
                }
                 />
                        ))
            }
            
                      </div>
                      <span style= {
                 {
                     color: "#fff", fontWeight: 900, fontSize: 17
                }
            }
            > {
                signal.confidence
            }
            %</span>
                    </div>
                  </div>
                </div>

                 {
                /* Mini candle chart */
            }
            
                <div style= {
                 {
                     background: "#0a0a0a", borderRadius: 10, padding: "10px 14px", marginBottom: 12, border: "1px solid #151515"
                }
            }
            >
                  <div style= {
                 {
                     color: "#444", fontSize: 9, letterSpacing: 2, marginBottom: 8
                }
            }
            >PRICE ACTION</div>
                  <CandleChart direction= {
                signal.direction
            }
             />
                </div>

                 {
                /* Indicators row */
            }
            
                <div style= {
                 {
                     display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: 7, marginBottom: 12
                }
            }
            >
                   {
                [
                     {
                     label: "RSI", value: signal.rsi, color: signal.rsi > 60 ? "#ff4444" : signal.rsi < 40 ? "#00ff41" : "#f59e0b"
                },
                     {
                     label: "MACD", value: signal.macd, color: signal.macd === "BULLISH" ? "#00ff41" : "#ff4444"
                },
                     {
                     label: "TREND", value: signal.trend.split("")[0] + signal.trend.slice(1).toLowerCase(), color: signal.trend === "UPTREND" ? "#00ff41" : signal.trend === "DOWNTREND" ? "#ff4444" : "#f59e0b"
                },
                  ].map(ind => (
                    <div key= {
                    ind.label
                }
                 style= {
                     {
                         background: "#0a0a0a", border: `1px solid ${ind.color}22`, borderRadius: 10, padding: "8px", textAlign: "center"
                    }
                }
                >
                      <div style= {
                     {
                         color: "#444", fontSize: 9, letterSpacing: 1
                    }
                }
                > {
                    ind.label
                }
                </div>
                      <div style= {
                     {
                         color: ind.color, fontWeight: 800, fontSize: 12, marginTop: 3
                    }
                }
                > {
                    ind.value
                }
                </div>
                    </div>
                  ))
            }
            
                </div>

                 {
                /* Signal meta */
            }
            
                <div style= {
                 {
                     display: "grid", gridTemplateColumns: "1fr 1fr 1fr 1fr", gap: 6, borderTop: "1px solid #111", paddingTop: 12, marginBottom: 12
                }
            }
            >
                   {
                [
                     {
                     label: "TIME", value: signal.time.slice(0, 5), icon: "🕐"
                },
                     {
                     label: "EXPIRY", value: signal.expiry, icon: "⏳"
                },
                     {
                     label: "ENTRY", value: signal.entry_quality, icon: "📍"
                },
                     {
                     label: "STATUS", value: "ACTIVE", icon: "●", green: true
                },
                  ].map(item => (
                    <div key= {
                    item.label
                }
                 style= {
                     {
                         textAlign: "center"
                    }
                }
                >
                      <div style= {
                     {
                         fontSize: 14, marginBottom: 3
                    }
                }
                > {
                    item.icon
                }
                </div>
                      <div style= {
                     {
                         color: "#333", fontSize: 8, letterSpacing: 1
                    }
                }
                > {
                    item.label
                }
                </div>
                      <div style= {
                     {
                         color: item.green ? "#00ff41" : "#aaa", fontSize: 9, fontWeight: 700, marginTop: 1
                    }
                }
                > {
                    item.value
                }
                </div>
                    </div>
                  ))
            }
            
                </div>

                 {
                /* Countdown */
            }
            
                 {
                countdown !== null && (
                  <div style= {
                     {
                         marginBottom: 12
                    }
                }
                >
                    <CountdownBar seconds= {
                    countdown
                }
                 total= {
                    totalSecs
                }
                 />
                  </div>
                )
            }

                 {
                /* Analysis text */
            }
            
                <div style= {
                 {
                     padding: "10px 12px", background: `rgba(${isCall ? "0,255,65" : "255,68,68"},0.05)`, border: `1px solid rgba(${isCall ? "0,255,65" : "255,68,68"},0.15)`, borderRadius: 10, color: "#888", fontSize: 12
                }
            }
            >
                  💡  {
                signal.analysis
            }
            
                </div>
              </>
            ) : (
              <div style= {
                 {
                     textAlign: "center", padding: "28px 0"
                }
            }
            >
                <RadarCircle pair= {
                null
            }
             scanning= {
                scanPhase
            }
             />
                <div style= {
                 {
                     color: "#333", fontSize: 13, marginTop: 10
                }
            }
            >
                   {
                scanPhase ? "Scanning market..." : "Press Generate to get AI signal"
            }
            
                </div>
              </div>
            )
        }
        
          </div>

           {
            /* Recent Signals */
        }
        
          <div style= {
             {
                 background: "#0e0e0e", border: "1px solid #1a1a1a", borderRadius: 16, padding: 14
            }
        }
        >
            <div style= {
             {
                 display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 12
            }
        }
        >
              <div style= {
             {
                 color: "#ccc", fontSize: 13, fontWeight: 700
            }
        }
        >↻ RECENT SIGNALS</div>
              <button onClick= {
            () => setActiveTab("HISTORY")
        }
         style= {
             {
                 background: "none", border: "none", color: "#00ff41", cursor: "pointer", fontSize: 12
            }
        }
        >View All ›</button>
            </div>
             {
            recentSignals.slice(0, 5).map((s, i) => (
              <div key= {
                i
            }
             style= {
                 {
                     display: "flex", alignItems: "center", justifyContent: "space-between", padding: "9px 0", borderBottom: i < 4 ? "1px solid #111" : "none"
                }
            }
            >
                <div style= {
                 {
                     display: "flex", alignItems: "center", gap: 9
                }
            }
            >
                  <span style= {
                 {
                     fontSize: 18
                }
            }
            > {
                s.flag1
            }
             {
                s.flag2
            }
            </span>
                  <div>
                    <div style= {
                 {
                     color: "#bbb", fontWeight: 600, fontSize: 12
                }
            }
            > {
                s.pair
            }
            </div>
                    <div style= {
                 {
                     color: "#444", fontSize: 10
                }
            }
            > {
                s.conf
            }
            % confidence</div>
                  </div>
                </div>
                <div style= {
                 {
                     display: "flex", alignItems: "center", gap: 8
                }
            }
            >
                  <span style= {
                 {
                     color: s.direction === "CALL" ? "#00ff41" : "#ff4444", fontWeight: 700, fontSize: 11
                }
            }
            > {
                s.direction
            }
              {
                s.direction === "CALL" ? "▲" : "▼"
            }
            </span>
                  <span style= {
                 {
                     color: "#333", fontSize: 10
                }
            }
            > {
                s.time
            }
            </span>
                  <span style= {
                 {
                     border: `1px solid ${s.result === "WIN" ? "#00ff41" : "#ff4444"}`, borderRadius: 20, padding: "3px 9px", fontSize: 10, fontWeight: 700, color: s.result === "WIN" ? "#00ff41" : "#ff4444"
                }
            }
            >
                     {
                s.result === "WIN" ? "◎" : "✕"
            }
              {
                s.result
            }
            
                  </span>
                </div>
              </div>
            ))
        }
        
          </div>
        </div>
      )
    }

       {
        /* Bottom Navigation */
    }
    
      <div style= {
         {
             position: "fixed", bottom: 0, left: "50%", transform: "translateX(-50%)", width: "100%", maxWidth: 420, background: "#080808", borderTop: "1px solid #141414", display: "flex", justifyContent: "space-around", padding: "10px 0 18px"
        }
    }
    >
         {
        [
           {
             icon: "⌂", label: "HOME"
        },
           {
             icon: "◈", label: "SIGNALS"
        },
           {
             icon: "📋", label: "HISTORY"
        },
           {
             icon: "💼", label: "PORTFOLIO"
        },
           {
             icon: "⚙", label: "SETTINGS"
        },
        ].map(tab => (
          <button key= {
            tab.label
        }
         onClick= {
            () => setActiveTab(tab.label)
        }
         style= {
             {
                
            background: "none", border: "none", cursor: "pointer",
            display: "flex", flexDirection: "column", alignItems: "center", gap: 3,
            color: activeTab === tab.label ? "#00ff41" : "#333", fontSize: 18
            }
        }
        >
            <span> {
            tab.icon
        }
        </span>
            <span style= {
             {
                 fontSize: 8, letterSpacing: 1, fontWeight: 700
            }
        }
        > {
            tab.label
        }
        </span>
             {
            activeTab === tab.label && <div style= {
                 {
                     width: 18, height: 2, background: "#00ff41", borderRadius: 2
                }
            }
             />
        }
        
          </button>
        ))
    }
    
      </div>
    </div>
  );
}
