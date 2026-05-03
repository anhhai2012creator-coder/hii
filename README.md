import React, { useMemo, useState } from "react";
import { motion, AnimatePresence } from "framer-motion";
import {
  Moon,
  Sun,
  Users,
  Bot,
  Copy,
  Play,
  Shield,
  Skull,
  Eye,
  HeartPulse,
  Crown,
  MessageCircle,
  Wifi,
  UserPlus,
  RefreshCcw,
  Sparkles,
} from "lucide-react";
import "./App.css";

const ROLE_DATA = {
  wolf: {
    name: "Ma sói",
    team: "Sói",
    icon: "🐺",
    desc: "Mỗi đêm cùng bầy sói chọn một người để cắn.",
  },
  villager: {
    name: "Dân làng",
    team: "Làng",
    icon: "🧑‍🌾",
    desc: "Không có kỹ năng ban đêm, thắng bằng suy luận và bỏ phiếu đúng.",
  },
  seer: {
    name: "Tiên tri",
    team: "Làng",
    icon: "🔮",
    desc: "Mỗi đêm soi một người để biết họ thuộc phe Sói hay không.",
  },
  doctor: {
    name: "Bảo vệ",
    team: "Làng",
    icon: "🛡️",
    desc: "Mỗi đêm bảo vệ một người khỏi bị sói cắn.",
  },
  witch: {
    name: "Phù thủy",
    team: "Làng",
    icon: "🧪",
    desc: "Có một bình cứu. Bản đơn giản cho phép cứu nạn nhân đêm đó.",
  },
  hunter: {
    name: "Thợ săn",
    team: "Làng",
    icon: "🏹",
    desc: "Khi chết có thể kéo theo một người. Bản này để vai trò hiển thị trước.",
  },
  cupid: {
    name: "Thần tình yêu",
    team: "Làng",
    icon: "💘",
    desc: "Đêm đầu ghép đôi hai người. Bản này để vai trò hiển thị trước.",
  },
  guard: {
    name: "Hiệp sĩ",
    team: "Làng",
    icon: "⚔️",
    desc: "Có tiếng nói mạnh khi tranh luận. Bản này tính phiếu như người thường.",
  },
  alphaWolf: {
    name: "Sói đầu đàn",
    team: "Sói",
    icon: "👑🐺",
    desc: "Sói đặc biệt, cùng phe Sói đi cắn mỗi đêm.",
  },
};

const BOT_NAMES = [
  "An Bot",
  "Bình Bot",
  "Chi Bot",
  "Dũng Bot",
  "Em Bot",
  "Giang Bot",
  "Hạ Bot",
  "Khoa Bot",
  "Linh Bot",
  "Minh Bot",
  "Nhi Bot",
  "Phong Bot",
  "Quân Bot",
  "Trang Bot",
  "Vy Bot",
  "Long Bot",
  "Tú Bot",
  "Huy Bot",
];

const ROLE_SET_BY_SIZE = {
  5: ["wolf", "seer", "doctor", "villager", "villager"],
  6: ["wolf", "seer", "doctor", "hunter", "villager", "villager"],
  7: ["wolf", "wolf", "seer", "doctor", "hunter", "villager", "villager"],
  8: ["alphaWolf", "wolf", "seer", "doctor", "witch", "hunter", "villager", "villager"],
  9: ["alphaWolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "villager", "villager"],
  10: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "villager", "villager"],
  11: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "guard", "villager", "villager"],
  12: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "guard", "villager", "villager", "villager"],
};

function makeCode() {
  return Math.random().toString(36).slice(2, 6).toUpperCase() + "-" + Math.random().toString(36).slice(2, 6).toUpperCase();
}

function shuffle(arr) {
  return [...arr].sort(() => Math.random() - 0.5);
}

function chooseRandom(list) {
  if (!list.length) return null;
  return list[Math.floor(Math.random() * list.length)];
}

function getRoleSet(count) {
  const safeCount = Math.max(5, Math.min(12, count));
  return ROLE_SET_BY_SIZE[safeCount] || ROLE_SET_BY_SIZE[12];
}

function createPlayer(name, isBot = false, host = false) {
  return {
    id: crypto.randomUUID(),
    name,
    isBot,
    host,
    alive: true,
    role: null,
  };
}

function Button({ children, onClick, variant = "primary", disabled = false }) {
  return (
    <button className={`btn ${variant}`} onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}

function Card({ children, className = "" }) {
  return <section className={`card ${className}`}>{children}</section>;
}

export default function App() {
  const [roomCode, setRoomCode] = useState(makeCode());
  const [playerName, setPlayerName] = useState("Chủ phòng");
  const [joinName, setJoinName] = useState("");
  const [joinCode, setJoinCode] = useState("");
  const [minPlayers, setMinPlayers] = useState(8);
  const [players, setPlayers] = useState([createPlayer("Chủ phòng", false, true)]);
  const [phase, setPhase] = useState("lobby");
  const [day, setDay] = useState(0);
  const [selectedWolfTarget, setSelectedWolfTarget] = useState("");
  const [selectedDoctorTarget, setSelectedDoctorTarget] = useState("");
  const [selectedSeerTarget, setSelectedSeerTarget] = useState("");
  const [selectedWitchSave, setSelectedWitchSave] = useState(false);
  const [seerResult, setSeerResult] = useState(null);
  const [nightVictim, setNightVictim] = useState(null);
  const [votes, setVotes] = useState({});
  const [log, setLog] = useState(["Phòng đã được tạo. Gửi mã phòng cho bạn bè để vào chơi."]);
  const [chat, setChat] = useState([{ from: "Hệ thống", text: "Chào mừng đến với Ma Sói!" }]);
  const [chatText, setChatText] = useState("");
  const [copied, setCopied] = useState(false);

  const alivePlayers = players.filter((p) => p.alive);
  const wolves = players.filter((p) => p.alive && ["wolf", "alphaWolf"].includes(p.role));
  const villagers = players.filter((p) => p.alive && !["wolf", "alphaWolf"].includes(p.role));

  const winner = useMemo(() => {
    if (phase === "lobby") return null;
    if (wolves.length === 0) return "Làng thắng! Tất cả ma sói đã bị loại.";
    if (wolves.length >= villagers.length) return "Sói thắng! Số sói đã bằng hoặc vượt số dân làng.";
    return null;
  }, [wolves.length, villagers.length, phase]);

  const roleSummary = useMemo(() => {
    const set = getRoleSet(minPlayers);
    const counted = set.reduce((acc, role) => {
      acc[role] = (acc[role] || 0) + 1;
      return acc;
    }, {});
    return Object.entries(counted);
  }, [minPlayers]);

  function pushLog(text) {
    setLog((prev) => [text, ...prev].slice(0, 14));
  }

  function resetGame() {
    setRoomCode(makeCode());
    setPlayers([createPlayer(playerName || "Chủ phòng", false, true)]);
    setPhase("lobby");
    setDay(0);
    setSelectedWolfTarget("");
    setSelectedDoctorTarget("");
    setSelectedSeerTarget("");
    setSelectedWitchSave(false);
    setSeerResult(null);
    setNightVictim(null);
    setVotes({});
    setLog(["Phòng mới đã được tạo."]);
    setChat([{ from: "Hệ thống", text: "Phòng mới đã sẵn sàng." }]);
  }

  function copyInvite() {
    const invite = `Mời bạn vào chơi Ma Sói! Mã phòng: ${roomCode}. Mở cùng link web rồi nhập mã để tham gia.`;
    navigator.clipboard?.writeText(invite);
    setCopied(true);
    setTimeout(() => setCopied(false), 1200);
  }

  function addHuman() {
    const name = joinName.trim();
    if (!name) return;
    setPlayers((prev) => [...prev, createPlayer(name, false)].slice(0, 12));
    setJoinName("");
    pushLog(`${name} đã vào phòng bằng mã ${joinCode || roomCode}.`);
  }

  function addBot(count = 1) {
    setPlayers((prev) => {
      const used = new Set(prev.map((p) => p.name));
      const fresh = [];
      for (let i = 0; i < count; i++) {
        const name = BOT_NAMES.find((n) => !used.has(n)) || `Bot ${prev.length + i + 1}`;
        used.add(name);
        fresh.push(createPlayer(name, true));
      }
      return [...prev, ...fresh].slice(0, 12);
    });
    pushLog(`Đã thêm ${count} người máy vào phòng.`);
  }

  function fillBots() {
    const need = Math.max(0, minPlayers - players.length);
    if (need > 0) addBot(need);
    else pushLog("Số người đã đủ, không cần thêm bot.");
  }

  function startGame() {
    let roster = [...players];
    const need = Math.max(0, minPlayers - roster.length);

    for (let i = 0; i < need; i++) {
      const name = BOT_NAMES.find((n) => !roster.some((p) => p.name === n)) || `Bot ${roster.length + 1}`;
      roster.push(createPlayer(name, true));
    }

    roster = roster.slice(0, 12);
    const roles = shuffle(getRoleSet(roster.length));
    const assigned = shuffle(roster).map((p, i) => ({
      ...p,
      role: roles[i],
      alive: true,
    }));

    setPlayers(assigned);
    setPhase("night");
    setDay(1);
    setVotes({});
    setNightVictim(null);
    setSeerResult(null);
    pushLog(`Game bắt đầu với ${assigned.length} người. Đêm 1 bắt đầu.`);
  }

  function autoNightChoices() {
    const possibleVictims = alivePlayers.filter((p) => !["wolf", "alphaWolf"].includes(p.role));
    const wolfTarget = selectedWolfTarget || chooseRandom(possibleVictims)?.id || "";
    const doctorTarget = selectedDoctorTarget || chooseRandom(alivePlayers)?.id || "";
    const seerTarget = selectedSeerTarget || chooseRandom(alivePlayers.filter((p) => p.role !== "seer"))?.id || "";

    setSelectedWolfTarget(wolfTarget);
    setSelectedDoctorTarget(doctorTarget);
    setSelectedSeerTarget(seerTarget);
    pushLog("Bot và các vai chưa chọn đã tự động hành động trong đêm.");
  }

  function resolveNight() {
    const wolfTarget = players.find((p) => p.id === selectedWolfTarget);
    const doctorTarget = players.find((p) => p.id === selectedDoctorTarget);
    const seerTarget = players.find((p) => p.id === selectedSeerTarget);

    let updated = players;
    let victim = null;

    if (wolfTarget) {
      const savedByDoctor = doctorTarget?.id === wolfTarget.id;
      const savedByWitch = selectedWitchSave;

      if (!savedByDoctor && !savedByWitch) {
        victim = wolfTarget;
        updated = players.map((p) => (p.id === wolfTarget.id ? { ...p, alive: false } : p));
      }
    }

    if (seerTarget) {
      const isWolf = ["wolf", "alphaWolf"].includes(seerTarget.role);
      setSeerResult(`${seerTarget.name} ${isWolf ? "thuộc phe Sói" : "không thuộc phe Sói"}.`);
    }

    setPlayers(updated);
    setNightVictim(victim ? `${victim.name} đã bị sói cắn.` : "Không ai chết trong đêm nay.");
    setPhase("day");
    setVotes({});
    pushLog(victim ? `Sáng ngày ${day}: ${victim.name} đã chết.` : `Sáng ngày ${day}: không ai chết.`);
  }

  function castVote(voterId, targetId) {
    setVotes((prev) => ({ ...prev, [voterId]: targetId }));
  }

  function autoVote() {
    const nextVotes = { ...votes };

    alivePlayers.forEach((voter) => {
      if (!nextVotes[voter.id]) {
        const candidates = alivePlayers.filter((p) => p.id !== voter.id);
        const suspicious = candidates.filter((p) => ["wolf", "alphaWolf"].includes(p.role));
        const pick = voter.isBot && Math.random() > 0.45 ? chooseRandom(candidates) : chooseRandom(suspicious.length ? suspicious : candidates);
        if (pick) nextVotes[voter.id] = pick.id;
      }
    });

    setVotes(nextVotes);
    pushLog("Các phiếu còn thiếu đã được tự động bỏ.");
  }

  function resolveVote() {
    const count = {};
    Object.values(votes).forEach((targetId) => {
      count[targetId] = (count[targetId] || 0) + 1;
    });

    const sorted = Object.entries(count).sort((a, b) => b[1] - a[1]);

    if (!sorted.length) {
      pushLog("Không đủ phiếu, không ai bị treo cổ.");
      setPhase("night");
      setDay((d) => d + 1);
      return;
    }

    const [targetId, amount] = sorted[0];
    const eliminated = players.find((p) => p.id === targetId);

    setPlayers((prev) => prev.map((p) => (p.id === targetId ? { ...p, alive: false } : p)));
    setPhase("night");
    setDay((d) => d + 1);
    setSelectedWolfTarget("");
    setSelectedDoctorTarget("");
    setSelectedSeerTarget("");
    setSelectedWitchSave(false);
    setNightVictim(null);
    pushLog(`${eliminated?.name || "Một người"} bị treo cổ với ${amount} phiếu. Đêm tiếp theo bắt đầu.`);
  }

  function sendChat() {
    const text = chatText.trim();
    if (!text) return;
    setChat((prev) => [...prev.slice(-7), { from: playerName || "Người chơi", text }]);
    setChatText("");
  }

  return (
    <main className="page">
      <div className="container">
        <motion.header initial={{ opacity: 0, y: -16 }} animate={{ opacity: 1, y: 0 }} className="hero">
          <div>
            <div className="badge"><Moon size={16} /> Ma Sói Web Demo</div>
            <h1>Trò chơi Ma Sói nhiều vai trò</h1>
            <p>Tạo phòng, chia mã mời bạn bè, tự thêm bot khi thiếu người và chơi theo vòng Đêm - Ngày - Bỏ phiếu.</p>
          </div>
          <div className="heroActions">
            <Button onClick={resetGame} variant="secondary"><RefreshCcw size={16} /> Phòng mới</Button>
            <Button onClick={copyInvite}><Copy size={16} /> {copied ? "Đã copy" : "Copy lời mời"}</Button>
          </div>
        </motion.header>

        <div className="layout">
          <div className="mainColumn">
            <Card>
              <div className="statsGrid">
                <div className="statBox">
                  <span><Wifi size={16} /> Mã phòng</span>
                  <strong>{roomCode}</strong>
                  <small>Gửi mã này cho bạn bè. Bản Vercel-only chỉ mô phỏng nhập phòng trên cùng trình duyệt.</small>
                </div>
                <div className="statBox">
                  <span><Users size={16} /> Người chơi</span>
                  <strong>{players.length}/{minPlayers}</strong>
                  <small>Tối thiểu 5, tối đa 12. Thiếu người thì game tự thêm bot.</small>
                </div>
                <div className="statBox">
                  <span>{phase === "night" ? <Moon size={16} /> : <Sun size={16} />} Trạng thái</span>
                  <strong>{phase === "lobby" ? "Sảnh" : phase === "night" ? `Đêm ${day}` : `Ngày ${day}`}</strong>
                  <small>{winner || "Game tiếp tục cho đến khi Làng hoặc Sói thắng."}</small>
                </div>
              </div>
            </Card>

            {phase === "lobby" && (
              <Card>
                <div className="sectionHeader">
                  <div>
                    <h2><UserPlus size={24} /> Sảnh chờ</h2>
                    <p>Thêm người thật, bot, chọn số người tối thiểu rồi bắt đầu.</p>
                  </div>
                  <Button onClick={startGame} variant="success"><Play size={16} /> Bắt đầu</Button>
                </div>

                <div className="formGrid">
                  <input value={playerName} onChange={(e) => setPlayerName(e.target.value)} placeholder="Tên của bạn" />
                  <input value={joinCode} onChange={(e) => setJoinCode(e.target.value.toUpperCase())} placeholder="Nhập mã phòng" />
                  <input value={joinName} onChange={(e) => setJoinName(e.target.value)} placeholder="Tên người chơi mới" />
                  <Button onClick={addHuman} variant="secondary">Thêm người chơi</Button>
                </div>

                <div className="toolRow">
                  <label>Số người mục tiêu</label>
                  <input type="range" min="5" max="12" value={minPlayers} onChange={(e) => setMinPlayers(Number(e.target.value))} />
                  <b>{minPlayers}</b>
                  <Button onClick={() => addBot(1)} variant="ghost"><Bot size={16} /> Thêm 1 bot</Button>
                  <Button onClick={fillBots} variant="ghost">Tự thêm cho đủ</Button>
                </div>

                <h3 className="miniTitle"><Sparkles size={18} /> Bộ vai trò khi đủ {minPlayers} người</h3>
                <div className="roleGrid">
                  {roleSummary.map(([role, count]) => (
                    <div key={role} className="roleMini">
                      <span>{ROLE_DATA[role].icon}</span>
                      <b>{ROLE_DATA[role].name} x{count}</b>
                      <small>Phe {ROLE_DATA[role].team}</small>
                    </div>
                  ))}
                </div>
              </Card>
            )}

            {phase !== "lobby" && (
              <Card>
                <AnimatePresence mode="wait">
                  {winner ? (
                    <motion.div key="winner" initial={{ opacity: 0, scale: 0.96 }} animate={{ opacity: 1, scale: 1 }} className="winnerBox">
                      <h2>{winner}</h2>
                      <p>Bấm “Phòng mới” để chơi ván khác.</p>
                    </motion.div>
                  ) : phase === "night" ? (
                    <motion.div key="night" initial={{ opacity: 0, y: 12 }} animate={{ opacity: 1, y: 0 }}>
                      <h2><Moon size={24} /> Pha đêm</h2>
                      <div className="actionGrid">
                        <ActionSelect title="Sói chọn cắn" icon={<Skull size={20} />} value={selectedWolfTarget} onChange={setSelectedWolfTarget} players={alivePlayers.filter((p) => !["wolf", "alphaWolf"].includes(p.role))} />
                        <ActionSelect title="Bảo vệ chọn cứu" icon={<Shield size={20} />} value={selectedDoctorTarget} onChange={setSelectedDoctorTarget} players={alivePlayers} />
                        <ActionSelect title="Tiên tri chọn soi" icon={<Eye size={20} />} value={selectedSeerTarget} onChange={setSelectedSeerTarget} players={alivePlayers} />
                      </div>

                      <label className="checkboxLine">
                        <input type="checkbox" checked={selectedWitchSave} onChange={(e) => setSelectedWitchSave(e.target.checked)} />
                        <HeartPulse size={20} /> Phù thủy dùng bình cứu nạn nhân đêm nay
                      </label>

                      <div className="toolRow">
                        <Button onClick={autoNightChoices} variant="secondary"><Bot size={16} /> Tự chọn hành động</Button>
                        <Button onClick={resolveNight}><Sun size={16} /> Kết thúc đêm</Button>
                      </div>
                    </motion.div>
                  ) : (
                    <motion.div key="day" initial={{ opacity: 0, y: 12 }} animate={{ opacity: 1, y: 0 }}>
                      <h2><Sun size={24} /> Pha ngày</h2>
                      <div className="infoGrid">
                        <div className="infoBox">
                          <b>Thông báo buổi sáng</b>
                          <p>{nightVictim || "Mọi người thức dậy và bắt đầu thảo luận."}</p>
                          {seerResult && <p className="seerResult">Kết quả riêng của Tiên tri: {seerResult}</p>}
                        </div>
                        <div className="infoBox">
                          <b>Bỏ phiếu treo cổ</b>
                          <p>Chọn mục tiêu cho từng người còn sống, hoặc để bot tự bỏ phiếu.</p>
                        </div>
                      </div>

                      <div className="voteList">
                        {alivePlayers.map((voter) => (
                          <div key={voter.id} className="voteRow">
                            <b>{voter.name} {voter.isBot && <small>BOT</small>}</b>
                            <select value={votes[voter.id] || ""} onChange={(e) => castVote(voter.id, e.target.value)}>
                              <option value="">Chọn người bị nghi</option>
                              {alivePlayers.filter((p) => p.id !== voter.id).map((p) => <option key={p.id} value={p.id}>{p.name}</option>)}
                            </select>
                          </div>
                        ))}
                      </div>

                      <div className="toolRow">
                        <Button onClick={autoVote} variant="secondary"><Bot size={16} /> Bot tự bỏ phiếu</Button>
                        <Button onClick={resolveVote} variant="danger"><Skull size={16} /> Chốt phiếu</Button>
                      </div>
                    </motion.div>
                  )}
                </AnimatePresence>
              </Card>
            )}

            <Card>
              <h2><Users size={24} /> Danh sách người chơi</h2>
              <div className="playerGrid">
                {players.map((p) => (
                  <motion.div layout key={p.id} className={`playerCard ${p.alive ? "" : "dead"}`}>
                    <div className="playerTop">
                      <div>
                        <b>{p.name}</b>
                        <small>{p.host && "Chủ phòng "}{p.isBot && "BOT "}{p.alive ? "Còn sống" : "Đã chết"}</small>
                      </div>
                      <span>{p.role ? ROLE_DATA[p.role].icon : p.isBot ? "🤖" : "🙂"}</span>
                    </div>
                    {p.role && (
                      <div className="roleBox">
                        <b>{ROLE_DATA[p.role].name}</b>
                        <small>Phe {ROLE_DATA[p.role].team}</small>
                        <p>{ROLE_DATA[p.role].desc}</p>
                      </div>
                    )}
                  </motion.div>
                ))}
              </div>
            </Card>
          </div>

          <aside className="sideColumn">
            <Card>
              <h2><Crown size={22} /> Luật thắng</h2>
              <p><b>Phe Làng:</b> thắng khi loại hết Ma Sói.</p>
              <p><b>Phe Sói:</b> thắng khi số Sói còn sống bằng hoặc nhiều hơn số dân làng.</p>
              <p><b>Bot:</b> tự hành động ban đêm và tự bỏ phiếu nếu người chơi chưa chọn.</p>
              <p><b>Vercel-only:</b> có link để mọi người mở game. Muốn nhiều máy đồng bộ thật thì sau này thêm Firebase/Supabase.</p>
            </Card>

            <Card>
              <h2><MessageCircle size={22} /> Chat thảo luận</h2>
              <div className="chatBox">
                {chat.map((m, idx) => (
                  <p key={idx}><b>{m.from}:</b> {m.text}</p>
                ))}
              </div>
              <div className="chatInput">
                <input value={chatText} onChange={(e) => setChatText(e.target.value)} onKeyDown={(e) => e.key === "Enter" && sendChat()} placeholder="Nhập tin nhắn..." />
                <Button onClick={sendChat}>Gửi</Button>
              </div>
            </Card>

            <Card>
              <h2>Nhật ký game</h2>
              <div className="logList">
                {log.map((entry, idx) => <div key={idx}>{entry}</div>)}
              </div>
            </Card>
          </aside>
        </div>
      </div>
    </main>
  );
}

function ActionSelect({ title, icon, value, onChange, players }) {
  return (
    <div className="actionBox">
      <b>{icon} {title}</b>
      <select value={value} onChange={(e) => onChange(e.target.value)}>
        <option value="">Chưa chọn</option>
        {players.map((p) => <option key={p.id} value={p.id}>{p.name}</option>)}
      </select>
    </div>
  );
}
