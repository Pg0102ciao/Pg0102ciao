# Sistema di Monitoraggio per Giardino Verticale Smart
# Questo script gestisce il monitoraggio dei sensori e controlla l'automazione

import time
import random
import json
from datetime import datetime
import matplotlib.pyplot as plt
import numpy as np
from collections import deque
import threading
import os

# In un sistema reale, queste librerie sarebbero utilizzate per interagire con hardware
# import RPi.GPIO as GPIO  # Per Raspberry Pi
# import Adafruit_DHT     # Per sensori di umidità/temperatura
# import board
# import busio
# import adafruit_ads1x15.ads1115 as ADS

class PlantModule:
    """Rappresenta un modulo singolo del giardino verticale"""
    
    def __init__(self, module_id, plant_type):
        self.module_id = module_id
        self.plant_type = plant_type
        self.moisture_data = deque(maxlen=100)
        self.light_data = deque(maxlen=100)
        self.temperature_data = deque(maxlen=100)
        self.last_watered = None
        
        # Valori ideali per questo tipo di pianta (da database o API)
        self.ideal_moisture = self._get_ideal_moisture(plant_type)
        self.ideal_light = self._get_ideal_light(plant_type)
        self.ideal_temperature = self._get_ideal_temperature(plant_type)
    
    def _get_ideal_moisture(self, plant_type):
        # In un sistema reale, questi dati verrebbero da un database
        plant_moisture = {
            "tomato": (60, 80),      # (min, max) percentuale
            "basil": (50, 70),
            "lettuce": (60, 75),
            "succulent": (20, 40),
            "strawberry": (55, 75)
        }
        return plant_moisture.get(plant_type.lower(), (50, 70))  # Default
    
    def _get_ideal_light(self, plant_type):
        # Valori in lux
        plant_light = {
            "tomato": (3000, 6000),
            "basil": (2500, 5000),
            "lettuce": (2000, 4000),
            "succulent": (4000, 8000),
            "strawberry": (3000, 5500)
        }
        return plant_light.get(plant_type.lower(), (2500, 5000))
    
    def _get_ideal_temperature(self, plant_type):
        # Valori in Celsius
        plant_temp = {
            "tomato": (18, 29),
            "basil": (18, 30),
            "lettuce": (15, 24),
            "succulent": (18, 32),
            "strawberry": (15, 26)
        }
        return plant_temp.get(plant_type.lower(), (18, 28))
    
    def read_sensors(self):
        """Legge i valori dai sensori (simulati)"""
        # In un sistema reale, qui leggeremmo dai sensori fisici
        moisture = random.uniform(self.ideal_moisture[0] - 20, self.ideal_moisture[1] + 10)
        light = random.uniform(self.ideal_light[0] - 1000, self.ideal_light[1] + 500)
        temperature = random.uniform(self.ideal_temperature[0] - 5, self.ideal_temperature[1] + 3)
        
        # Salva i dati
        timestamp = datetime.now()
        self.moisture_data.append((timestamp, moisture))
        self.light_data.append((timestamp, light))
        self.temperature_data.append((timestamp, temperature))
        
        return {
            "timestamp": timestamp.strftime("%Y-%m-%d %H:%M:%S"),
            "module_id": self.module_id,
            "plant_type": self.plant_type,
            "moisture": moisture,
            "light": light,
            "temperature": temperature,
            "moisture_status": self._check_moisture_status(moisture),
            "light_status": self._check_light_status(light),
            "temperature_status": self._check_temperature_status(temperature)
        }
    
    def _check_moisture_status(self, moisture):
        """Verifica se l'umidità è nell'intervallo ideale"""
        min_moisture, max_moisture = self.ideal_moisture
        if moisture < min_moisture:
            return "LOW"
        elif moisture > max_moisture:
            return "HIGH"
        else:
            return "OPTIMAL"
    
    def _check_light_status(self, light):
        """Verifica se la luce è nell'intervallo ideale"""
        min_light, max_light = self.ideal_light
        if light < min_light:
            return "LOW"
        elif light > max_light:
            return "HIGH"
        else:
            return "OPTIMAL"
    
    def _check_temperature_status(self, temperature):
        """Verifica se la temperatura è nell'intervallo ideale"""
        min_temp, max_temp = self.ideal_temperature
        if temperature < min_temp:
            return "LOW"
        elif temperature > max_temp:
            return "HIGH"
        else:
            return "OPTIMAL"
    
    def water_plants(self, amount=100):
        """Attiva il sistema di irrigazione"""
        self.last_watered = datetime.now()
        print(f"Watering module {self.module_id} with {amount}ml of water")
        # In un sistema reale, qui attiveremmo una pompa
        # GPIO.output(PUMP_PIN, GPIO.HIGH)
        # time.sleep(amount / 100)  # Tempo proporzionale alla quantità d'acqua
        # GPIO.output(PUMP_PIN, GPIO.LOW)
        return {
            "status": "success",
            "message": f"Module {self.module_id} watered with {amount}ml",
            "timestamp": self.last_watered.strftime("%Y-%m-%d %H:%M:%S")
        }
    
    def adjust_light(self, intensity):
        """Regola l'intensità della luce LED"""
        print(f"Adjusting light in module {self.module_id} to {intensity}%")
        # In un sistema reale, regoleremmo i LED
        # pwm.ChangeDutyCycle(intensity)
        return {
            "status": "success",
            "message": f"Light intensity set to {intensity}%"
        }
    
    def generate_report(self):
        """Genera un report sullo stato del modulo"""
        if not self.moisture_data:
            return {"error": "No data available"}
        
        # Estrai gli ultimi valori
        last_moisture = self.moisture_data[-1][1]
        last_light = self.light_data[-1][1]
        last_temp = self.temperature_data[-1][1]
        
        # Calcola medie
        avg_moisture = sum(m[1] for m in self.moisture_data) / len(self.moisture_data)
        avg_light = sum(l[1] for l in self.light_data) / len(self.light_data)
        avg_temp = sum(t[1] for t in self.temperature_data) / len(self.temperature_data)
        
        return {
            "module_id": self.module_id,
            "plant_type": self.plant_type,
            "current_status": {
                "moisture": last_moisture,
                "moisture_status": self._check_moisture_status(last_moisture),
                "light": last_light,
                "light_status": self._check_light_status(last_light),
                "temperature": last_temp,
                "temperature_status": self._check_temperature_status(last_temp)
            },
            "averages": {
                "moisture": avg_moisture,
                "light": avg_light,
                "temperature": avg_temp
            },
            "last_watered": self.last_watered.strftime("%Y-%m-%d %H:%M:%S") if self.last_watered else "Never",
            "recommendations": self._generate_recommendations(last_moisture, last_light, last_temp)
        }
    
    def _generate_recommendations(self, moisture, light, temperature):
        """Genera raccomandazioni basate sui valori attuali"""
        recommendations = []
        
        # Umidità
        if moisture < self.ideal_moisture[0]:
            recommendations.append(f"Water the {self.plant_type} soon. Current moisture: {moisture:.1f}%")
        elif moisture > self.ideal_moisture[1]:
            recommendations.append(f"Reduce watering for the {self.plant_type}. Current moisture: {moisture:.1f}%")
        
        # Luce
        if light < self.ideal_light[0]:
            recommendations.append(f"Increase light exposure for the {self.plant_type}. Current light: {light:.0f} lux")
        elif light > self.ideal_light[1]:
            recommendations.append(f"Reduce light intensity for the {self.plant_type}. Current light: {light:.0f} lux")
        
        # Temperatura
        if temperature < self.ideal_temperature[0]:
            recommendations.append(f"Increase temperature for the {self.plant_type}. Current: {temperature:.1f}°C")
        elif temperature > self.ideal_temperature[1]:
            recommendations.append(f"Reduce temperature for the {self.plant_type}. Current: {temperature:.1f}°C")
        
        if not recommendations:
            recommendations.append(f"All parameters are optimal for {self.plant_type}")
            
        return recommendations
    
    def plot_data(self):
        """Crea grafici per visualizzare i dati storici"""
        if not self.moisture_data:
            print("No data available for plotting")
            return
        
        # Prepara i dati per i grafici
        timestamps = [m[0] for m in self.moisture_data]
        moisture_values = [m[1] for m in self.moisture_data]
        light_values = [l[1] for l in self.light_data]
        temp_values = [t[1] for t in self.temperature_data]
        
        # Crea una figura con tre sottografici
        fig, (ax1, ax2, ax3) = plt.subplots(3, 1, figsize=(10, 12))
        fig.suptitle(f'Module {self.module_id}: {self.plant_type.capitalize()} - Sensor Data', fontsize=16)
        
        # Grafico umidità
        ax1.plot(timestamps, moisture_values, 'b-')
        ax1.axhspan(self.ideal_moisture[0], self.ideal_moisture[1], alpha=0.2, color='green')
        ax1.set_ylabel('Moisture (%)')
        ax1.set_title('Soil Moisture')
        ax1.grid(True)
        
        # Grafico luce
        ax2.plot(timestamps, light_values, 'y-')
        ax2.axhspan(self.ideal_light[0], self.ideal_light[1], alpha=0.2, color='green')
        ax2.set_ylabel('Light (lux)')
        ax2.set_title('Light Intensity')
        ax2.grid(True)
        
        # Grafico temperatura
        ax3.plot(timestamps, temp_values, 'r-')
        ax3.axhspan(self.ideal_temperature[0], self.ideal_temperature[1], alpha=0.2, color='green')
        ax3.set_ylabel('Temperature (°C)')
        ax3.set_title('Temperature')
        ax3.grid(True)
        
        # Formatta l'asse x
        for ax in [ax1, ax2, ax3]:
            ax.set_xlim(min(timestamps), max(timestamps))
            plt.setp(ax.get_xticklabels(), rotation=45)
        
        plt.tight_layout(rect=[0, 0, 1, 0.95])
        
        # Salva e mostra il grafico
        filename = f'module_{self.module_id}_status.png'
        plt.savefig(filename)
        print(f"Plot saved as {filename}")
        print(f"File path: {os.path.abspath(filename)}")
        
        # Mostra il grafico invece di solo salvarlo
        plt.show()


class SmartGardenSystem:
    """Gestisce l'intero sistema di giardino verticale"""
    
    def __init__(self):
        self.modules = {}
        self.water_level = 100  # Percentuale
        self.system_status = "OK"
        self.notifications = []
        self.automation_active = True
    
    def add_module(self, module_id, plant_type):
        """Aggiunge un nuovo modulo al sistema"""
        self.modules[module_id] = PlantModule(module_id, plant_type)
        print(f"Module {module_id} with {plant_type} added to the system")
    
    def remove_module(self, module_id):
        """Rimuove un modulo dal sistema"""
        if module_id in self.modules:
            del self.modules[module_id]
            print(f"Module {module_id} removed from the system")
            return True
        return False
    
    def monitor_all(self):
        """Legge i sensori di tutti i moduli"""
        results = {}
        for module_id, module in self.modules.items():
            results[module_id] = module.read_sensors()
        
        # Aggiorna il livello dell'acqua (simulato)
        self.update_water_level()
        
        # Controlla se è necessario inviare notifiche
        self.check_for_alerts(results)
        
        return results
    
    def update_water_level(self):
        """Aggiorna il livello dell'acqua nel serbatoio (simulato)"""
        # In un sistema reale, leggeremmo da un sensore
        # Simuliamo una leggera diminuzione
        self.water_level -= random.uniform(0, 0.5)
        if self.water_level < 0:
            self.water_level = 0
        
        # Genera un alert se il livello è basso
        if self.water_level < 20 and "LOW_WATER" not in [n["type"] for n in self.notifications]:
            self.add_notification("LOW_WATER", f"Water level is low ({self.water_level:.1f}%). Please refill the reservoir.")
    
    def check_for_alerts(self, sensor_data):
        """Controlla se ci sono condizioni che richiedono notifiche"""
        for module_id, data in sensor_data.items():
            plant_type = data["plant_type"]
            
            # Controllo umidità
            if data["moisture_status"] == "LOW":
                self.add_notification(
                    "LOW_MOISTURE", 
                    f"Module {module_id} ({plant_type}): Soil moisture is low ({data['moisture']:.1f}%)"
                )
                
                # Se l'automazione è attiva e c'è abbastanza acqua, innaffia
                if self.automation_active and self.water_level > 10:
                    self.modules[module_id].water_plants()
                    # Riduce il livello dell'acqua
                    self.water_level -= 5
            
            # Controllo luce
            if data["light_status"] == "LOW":
                self.add_notification(
                    "LOW_LIGHT", 
                    f"Module {module_id} ({plant_type}): Light level is low ({data['light']:.0f} lux)"
                )
                
                # Se l'automazione è attiva, aumenta la luce
                if self.automation_active:
                    self.modules[module_id].adjust_light(90)
            
            # Controllo temperatura
            if data["temperature_status"] != "OPTIMAL":
                self.add_notification(
                    f"{data['temperature_status']}_TEMP", 
                    f"Module {module_id} ({plant_type}): Temperature is {data['temperature_status'].lower()} ({data['temperature']:.1f}°C)"
                )
    
    def add_notification(self, notification_type, message):
        """Aggiunge una notifica al sistema"""
        notification = {
            "type": notification_type,
            "message": message,
            "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            "read": False
        }
        self.notifications.append(notification)
        print(f"NOTIFICATION: {message}")
    
    def get_notifications(self, unread_only=False):
        """Recupera le notifiche"""
        if unread_only:
            return [n for n in self.notifications if not n["read"]]
        return self.notifications
    
    def mark_notification_read(self, index):
        """Segna una notifica come letta"""
        if 0 <= index < len(self.notifications):
            self.notifications[index]["read"] = True
            return True
        return False
    
    def mark_all_notifications_read(self):
        """Segna tutte le notifiche come lette"""
        for notification in self.notifications:
            notification["read"] = True
    
    def get_system_status(self):
        """Restituisce lo stato dell'intero sistema"""
        return {
            "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            "system_status": self.system_status,
            "water_level": self.water_level,
            "modules": len(self.modules),
            "automation_active": self.automation_active,
            "unread_notifications": len([n for n in self.notifications if not n["read"]])
        }
    
    def generate_full_report(self):
        """Genera un report completo su tutto il sistema"""
        report = {
            "system": self.get_system_status(),
            "modules": {}
        }
        
        for module_id, module in self.modules.items():
            report["modules"][module_id] = module.generate_report()
        
        return report
    
    def set_automation(self, active):
        """Attiva o disattiva l'automazione"""
        self.automation_active = active
        status = "activated" if active else "deactivated"
        print(f"System automation {status}")
        return {
            "status": "success",
            "automation": status
        }
    
    def refill_water(self, amount=100):
        """Riempie il serbatoio dell'acqua"""
        old_level = self.water_level
        self.water_level = min(100, self.water_level + amount)
        print(f"Water reservoir refilled from {old_level:.1f}% to {self.water_level:.1f}%")
        
        # Rimuove le notifiche di acqua bassa
        self.notifications = [n for n in self.notifications if n["type"] != "LOW_WATER"]
        
        return {
            "status": "success",
            "old_level": old_level,
            "new_level": self.water_level
        }
    
    def simulate_day(self, days=1, reading_interval=3):
        """Simula il passaggio di giorni per test e demo"""
        total_readings = days * 24 * (60 // reading_interval)
        print(f"Simulating {days} day(s) with readings every {reading_interval} minutes ({total_readings} readings)")
        
        for i in range(total_readings):
            # Leggi sensori
            self.monitor_all()
            
            # Aggiungi un po' di variazione ai dati
            for module in self.modules.values():
                # Simula il ciclo giorno/notte per la luce
                hour = (i * reading_interval) % (24 * 60) // 60
                if 6 <= hour < 18:  # Giorno
                    module.ideal_light = (module._get_ideal_light(module.plant_type)[0], 
                                         module._get_ideal_light(module.plant_type)[1] * 1.2)
                else:  # Notte
                    module.ideal_light = (module._get_ideal_light(module.plant_type)[0] * 0.1,
                                         module._get_ideal_light(module.plant_type)[1] * 0.3)
            
            # Pausa tra le letture (accelerata per la simulazione)
            time.sleep(0.1)  # Invece di reading_interval minuti
            
            # Stampa progresso
            if i % 10 == 0:
                progress = i / total_readings * 100
                print(f"Simulation progress: {progress:.1f}%")
        
        print("Simulation complete")


# Dimostrazione del sistema
def run_demo():
    """Esegue una demo del sistema"""
    print("Initializing Smart Garden System...")
    garden = SmartGardenSystem()
    
    # Aggiunge moduli con piante diverse
    garden.add_module(1, "tomato")
    garden.add_module(2, "basil")
    garden.add_module(3, "succulent")
    
    print("\nInitial system status:")
    print(json.dumps(garden.get_system_status(), indent=2))
    
    print("\nSimulating 3 days of operation...")
    garden.simulate_day(days=3, reading_interval=60)
    
    print("\nGenerating reports and plots for each module...")
    for module_id, module in garden.modules.items():
        module.plot_data()  # Ora questo mostrerà i grafici oltre a salvarli
        report = module.generate_report()
        print(f"\nModule {module_id} ({report['plant_type']}) Report:")
        print(json.dumps(report['current_status'], indent=2))
        print("\nRecommendations:")
        for rec in report['recommendations']:
            print(f"- {rec}")
    
    print("\nSystem notifications:")
    for i, notification in enumerate(garden.get_notifications()):
        print(f"{i+1}. [{notification['timestamp']}] {notification['message']}")
    
    print("\nFull system report:")
    full_report = garden.generate_full_report()
    print(f"System status: {full_report['system']['system_status']}")
    print(f"Water level: {full_report['system']['water_level']:.1f}%")
    print(f"Modules: {full_report['system']['modules']}")
    print(f"Automation: {'Active' if full_report['system']['automation_active'] else 'Inactive'}")

    print("\nDemo complete")


if __name__ == "__main__":
    run_demo()
