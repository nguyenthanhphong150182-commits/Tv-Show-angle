const initialLayers = [
  { id: "bg", name: "Background", src: "/pngtuber/bg.png", x: 0, y: 0, scale: 1, visible: true, z: 0 },
  { id: "body", name: "Body", src: "/pngtuber/body.png", x: 0, y: 0, scale: 1, visible: true, z: 10 },
  { id: "eyes", name: "Eyes", src: "/pngtuber/eyes.png", x: 0, y: -10, scale: 1, visible: true, z: 20 },
  { id: "mouth", name: "Mouth", src: "/pngtuber/mouth.png", x: 0, y: 30, scale: 1, visible: true, z: 30 },
  { id: "acc", name: "Accessory", src: "/pngtuber/acc.png", x: 20, y: -20, scale: 1, visible: true, z: 40 },
];const initialLayers = [
  { id: "bg", name: "Background", src: "/pngtuber/bg.png", x: 0, y: 0, scale: 1, visible: true, z: 0 },
  { id: "body", name: "Body", src: "/pngtuber/body.png", x: 0, y: 0, scale: 1, visible: true, z: 10 },
  { id: "eyes", name: "Eyes", src: "/pngtuber/eyes.png", x: 0, y: -10, scale: 1, visible: true, z: 20 },
  { id: "mouth", name: "Mouth", src: "/pngtuber/mouth.png", x: 0, y: 30, scale: 1, visible: true, z: 30 },
  { id: "acc", name: "Accessory", src: "/pngtuber/acc.png", x: 20, y: -20, scale: 1, visible: true, z: 40 },
];public/pngtuber/body.png
public/pngtuber/eyes.png
public/pngtuber/mouth.png
public/pngtuber/acc.pngimport React, { useState, useRef, useEffect } from "react";
import { motion, useMotionValue } from "framer-motion";

export default function PNGtuber() {
  const initialLayers = [
    { id: "bg", name: "Background", src: null, x: 0, y: 0, scale: 1, visible: true, z: 0 },
    { id: "body", name: "Body", src: null, x: 0, y: 0, scale: 1, visible: true, z: 10 },
    { id: "eyes", name: "Eyes", src: null, x: 0, y: -10, scale: 1, visible: true, z: 20 },
    { id: "mouth", name: "Mouth", src: null, x: 0, y: 30, scale: 1, visible: true, z: 30 },
    { id: "acc", name: "Accessory", src: null, x: 20, y: -20, scale: 1, visible: true, z: 40 },
  ];

  const [layers, setLayers] = useState(initialLayers);
  const [selectedId, setSelectedId] = useState("body");
  const stageRef = useRef(null);

  function handleFileChange(e, layerId) {
    const file = e.target.files && e.target.files[0];
    if (!file) return;
    const url = URL.createObjectURL(file);
    setLayers((prev) => prev.map((l) => (l.id === layerId ? { ...l, src: url } : l)));
  }

  function toggleVisibility(id) {
    setLayers((prev) => prev.map((l) => (l.id === id ? { ...l, visible: !l.visible } : l)));
  }

  function updateLayer(id, patch) {
    setLayers((prev) => prev.map((l) => (l.id === id ? { ...l, ...patch } : l)));
  }

  function bringForward(id) {
    setLayers((prev) => {
      const maxZ = Math.max(...prev.map((p) => p.z));
      return prev.map((l) => (l.id === id ? { ...l, z: maxZ + 10 } : l));
    });
  }

  function saveScene() {
    const data = JSON.stringify(layers);
    const blob = new Blob([data], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "pngtuber-scene.json";
    a.click();
  }

  function loadScene(e) {
    const file = e.target.files && e.target.files[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = () => {
      try {
        const json = JSON.parse(reader.result);
        setLayers(json);
      } catch (err) {
        alert("Invalid scene file");
      }
    };
    reader.readAsText(file);
  }

  function LayerItem({ layer }) {
    const x = useMotionValue(layer.x);
    const y = useMotionValue(layer.y);

    useEffect(() => {
      x.set(layer.x);
      y.set(layer.y);
    }, [layer.x, layer.y]);

    return (
      <motion.img
        src={layer.src}
        alt={layer.name}
        style={{
          x,
          y,
          scale: layer.scale,
          zIndex: layer.z,
          position: "absolute",
          pointerEvents: layer.visible ? "auto" : "none",
        }}
        draggable={false}
        drag
        dragConstraints={stageRef}
        dragElastic={0.1}
        onDragEnd={(ev, info) => {
          const rect = stageRef.current.getBoundingClientRect();
          const newX = info.point.x - rect.left - rect.width / 2;
          const newY = info.point.y - rect.top - rect.height / 2;
          updateLayer(layer.id, { x: newX, y: newY });
        }}
        onDoubleClick={() => bringForward(layer.id)}
        className={`select-none ${layer.visible ? "" : "opacity-40"} cursor-grab`}
      />
    );
  }

  return (
    <div className="flex h-screen bg-gray-900 text-white">
      <div className="w-64 p-4 border-r border-gray-700 space-y-2 overflow-y-auto">
        <h2 className="text-lg font-bold mb-2">Layers</h2>
        {layers.map((l) => (
          <div
            key={l.id}
            className={`p-2 rounded ${
              selectedId === l.id ? "bg-gray-700" : "bg-gray-800"
            } flex justify-between items-center`}
          >
            <button onClick={() => setSelectedId(l.id)}>{l.name}</button>
            <button onClick={() => toggleVisibility(l.id)}>
              {l.visible ? "👁️" : "🚫"}
            </button>
          </div>
        ))}

        <div className="mt-4 space-y-2">
          <label>Upload PNG for {selectedId}</label>
          <input
            type="file"
            accept="image/png"
            onChange={(e) => handleFileChange(e, selectedId)}
          />

          <div className="flex gap-2 mt-4">
            <button className="bg-blue-600 px-2 py-1 rounded" onClick={saveScene}>
              Save
            </button>
            <label className="bg-green-600 px-2 py-1 rounded cursor-pointer">
              Load
              <input
                type="file"
                accept="application/json"
                className="hidden"
                onChange={loadScene}
              />
            </label>
          </div>
        </div>
      </div>

      <div
        ref={stageRef}
        className="flex-1 relative flex items-center justify-center bg-gray-800 overflow-hidden"
      >
        {layers
          .filter((l) => l.visible && l.src)
          .map((l) => (
            <LayerItem key={l.id} layer={l} />
          ))}
      </div>
    </div>
  );
}
