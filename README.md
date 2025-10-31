from flask import Flask, render_template, request, jsonify
import os

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/convert', methods=['POST'])
def convert_temperature():
    try:
        data = request.get_json()
        
        temperature = float(data.get('temperature'))
        from_unit = data.get('fromUnit')
        to_unit = data.get('toUnit')
        
        # Валидация входных данных
        if from_unit not in ['celsius', 'fahrenheit', 'kelvin'] or to_unit not in ['celsius', 'fahrenheit', 'kelvin']:
            return jsonify({'error': 'Неверные единицы измерения'}), 400
        
        # Конвертация через Celsius как промежуточную
        if from_unit == 'celsius':
            celsius = temperature
        elif from_unit == 'fahrenheit':
            celsius = (temperature - 32) * 5/9
        elif from_unit == 'kelvin':
            celsius = temperature - 273.15
        
        # Конвертация из Celsius в целевую единицу
        if to_unit == 'celsius':
            result = celsius
        elif to_unit == 'fahrenheit':
            result = (celsius * 9/5) + 32
        elif to_unit == 'kelvin':
            result = celsius + 273.15
        
        # Округление до 2 знаков после запятой
        result = round(result, 2)
        
        return jsonify({
            'result': result,
            'fromUnit': from_unit,
            'toUnit': to_unit,
            'originalTemp': temperature
        })
        
    except ValueError:
        return jsonify({'error': 'Пожалуйста, введите корректное числовое значение температуры'}), 400
    except Exception as e:
        return jsonify({'error': f'Ошибка при конвертации: {str(e)}'}), 500

@app.route('/health')
def health_check():
    return jsonify({'status': 'OK', 'message': 'Сервер конвертера температуры работает'})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
    
