# Bet-hub
import React, { useState, useEffect } from "react";
import { motion } from "framer-motion";

const cards = ["A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"];

function getRandomCard() {
  return cards[Math.floor(Math.random() * cards.length)];
}

function getCardValue(card) {
  if (card === "A") return 1;
  if (card === "J") return 11;
  if (card === "Q") return 12;
  if (card === "K") return 13;
  return parseInt(card);
}

export default function DragonTigerBet() {
  const [playerName] = useState("ผู้เล่น"); // ชื่อคงที่ ไม่แก้ได้
  const [credit, setCredit] = useState(() => parseInt(localStorage.getItem("credit")) || 10000);
  const [bet, setBet] = useState(1000);
  const [betSide, setBetSide] = useState(null); // 'dragon' หรือ 'tiger'
  const [dragonCard, setDragonCard] = useState(null);
  const [tigerCard, setTigerCard] = useState(null);
  const [result, setResult] = useState(null); // 'win', 'lose', 'tie'
  const [message, setMessage] = useState("");

  useEffect(() => {
    localStorage.setItem("credit", credit);
  }, [credit]);

  function playRound() {
    if (!betSide) {
      setMessage("โปรดเลือกเดิมพัน ฝั่งมังกร หรือ เสือ ก่อนครับ");
      return;
    }
    if (bet > credit) {
      setMessage("เครดิตไม่พอสำหรับเดิมพันนี้ครับ");
      return;
    }
    setMessage("");
    setResult(null);
    setDragonCard(null);
    setTigerCard(null);

    const dCard = getRandomCard();
    const tCard = getRandomCard();

    // เปิดไพ่มังกรก่อน
    setDragonCard(dCard);

    // เปิดไพ่เสือทีหลัง + ตัดสินผล
    setTimeout(() => {
      setTigerCard(tCard);

      const dVal = getCardValue(dCard);
      const tVal = getCardValue(tCard);

      let isWin = false;
      let isTie = false;

      if (dVal === tVal) {
        isTie = true;
        setResult("tie");
        setMessage("เสมอ ไม่ได้ไม่เสียเครดิต");
      } else {
        // ฝั่งที่แต้มมากกว่าชนะ
        let winner = dVal > tVal ? "dragon" : "tiger";
        if (betSide === winner) {
          isWin = true;
          setResult("win");
          setCredit(c => c + bet);
          setMessage(`ชนะ +${bet} บาท`);
        } else {
          setResult("lose");
          setCredit(c => c - bet);
          setMessage(`แพ้ -${bet} บาท`);
        }
      }
    }, 1500);
  }

  return (
    <div style={{ maxWidth: 360, margin: "auto", padding: 16, fontFamily: "Arial, sans-serif" }}>
      <div style={{ textAlign: "right", marginBottom: 8, fontWeight: "bold" }}>
        👤 {playerName}
      </div>

      <div style={{ display: "flex", justifyContent: "space-around", marginBottom: 16 }}>
        <div style={{ textAlign: "center" }}>
          <div style={{ fontWeight: "bold", marginBottom: 4 }}>มังกร</div>
          <motion.div
            key={dragonCard || "?"}
            initial={{ opacity: 0, y: -20 }}
            animate={{ opacity: 1, y: 0 }}
            style={{ fontSize: 48 }}
          >
            {dragonCard || "?"}
          </motion.div>
        </div>
        <div style={{ textAlign: "center" }}>
          <div style={{ fontWeight: "bold", marginBottom: 4 }}>เสือ</div>
          <motion.div
            key={tigerCard || "?"}
            initial={{ opacity: 0, y: -20 }}
            animate={{ opacity: 1, y: 0 }}
            style={{ fontSize: 48 }}
          >
            {tigerCard || "?"}
          </motion.div>
        </div>
      </div>

      <div style={{ marginBottom: 16, textAlign: "center" }}>
        <div style={{ marginBottom: 8, fontWeight: "bold" }}>เลือกเดิมพัน</div>
        <button
          onClick={() => setBetSide("dragon")}
          style={{
            marginRight: 12,
            padding: "8px 16px",
            backgroundColor: betSide === "dragon" ? "#0f0" : "#ccc",
            border: "none",
            borderRadius: 4,
            cursor: "pointer",
          }}
        >
          มังกร
        </button>
        <button
          onClick={() => setBetSide("tiger")}
          style={{
            padding: "8px 16px",
            backgroundColor: betSide === "tiger" ? "#0f0" : "#ccc",
            border: "none",
            borderRadius: 4,
            cursor: "pointer",
          }}
        >
          เสือ
        </button>
      </div>

      <div style={{ marginBottom: 16, textAlign: "center" }}>
        <label>
          เดิมพัน:{" "}
          <input
            type="number"
            value={bet}
            onChange={(e) => setBet(Math.min(20000, Math.max(1, Number(e.target.value))))}
            min={1}
            max={20000}
            style={{ width: 80, padding: 4, fontSize: 16 }}
          />{" "}
          บาท (สูงสุด 20,000)
        </label>
      </div>

      <div style={{ textAlign: "center", marginBottom: 16, fontWeight: "bold", fontSize: 18 }}>
        เครดิตคงเหลือ: ฿{credit}
      </div>

      <div style={{ textAlign: "center", marginBottom: 16, color: result === "win" ? "green" : result === "lose" ? "red" : "orange", fontWeight: "bold", fontSize: 20 }}>
        {message}
      </div>

      <div style={{ textAlign: "center" }}>
        <button
          onClick={playRound}
          disabled={credit < bet || !betSide}
          style={{
            backgroundColor: credit < bet || !betSide ? "#999" : "#007bff",
            color: "white",
            border: "none",
            padding: "12px 24px",
            borderRadius: 6,
            cursor: credit < bet || !betSide ? "not-allowed" : "pointer",
            fontSize: 16,
          }}
        >
          เปิดไพ่
        </button>
      </div>
    </div>
  );
}
