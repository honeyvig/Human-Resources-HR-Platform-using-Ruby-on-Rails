# Human-Resources-HR-Platform-using-Ruby-on-Rails
We are seeking a skilled developer to assist in building a web application for a Human Resources platform. The ideal candidate will have experience in full-stack web development and a strong understanding of HR systems isn't required but preferred. The project involves designing user interfaces, implementing backend functionalities, and ensuring the application meets user requirements. Collaboration with our team is essential to create an efficient and user-friendly platform. If you are passionate about developing impactful solutions in the HR space, we want to hear from you!
================
Here’s a foundational implementation for a Human Resources (HR) platform using Ruby on Rails. This project includes user management, employee data tracking, and role-based functionalities.
1. Create the Rails Application

Run the following to set up the application:

rails new hr_platform --database=postgresql
cd hr_platform

2. Set Up Models
User Model (for authentication and roles)

Generate the User model:

rails generate model User name:string email:string role:string password_digest:string

Employee Model (for managing employee data)

rails generate model Employee first_name:string last_name:string email:string phone:string position:string salary:decimal user:references

Other Models (e.g., Departments, LeaveRequests)

For department management:

rails generate model Department name:string description:string

For leave management:

rails generate model LeaveRequest employee:references start_date:date end_date:date status:string

3. Install Gems

Add the following gems to your Gemfile:

gem 'bcrypt', '~> 3.1.7' # For authentication
gem 'pundit' # For authorization
gem 'chartkick' # For analytics
gem 'groupdate'
gem 'bootstrap', '~> 5.0.0'

Run bundle install to install the gems.
4. Configure User Authentication

Set up has_secure_password in the User model:

class User < ApplicationRecord
  has_secure_password
  validates :email, presence: true, uniqueness: true
  enum role: { admin: 'admin', manager: 'manager', employee: 'employee' }
end

Create a SessionsController for login/logout:

rails generate controller Sessions

5. Define Relationships

Set up model associations:

class Employee < ApplicationRecord
  belongs_to :user
  belongs_to :department, optional: true
end

class Department < ApplicationRecord
  has_many :employees
end

class LeaveRequest < ApplicationRecord
  belongs_to :employee
end

6. Add Routes

Define routes in config/routes.rb:

Rails.application.routes.draw do
  root 'dashboard#index'

  resources :users
  resources :employees do
    resources :leave_requests
  end
  resources :departments

  get '/login', to: 'sessions#new'
  post '/login', to: 'sessions#create'
  delete '/logout', to: 'sessions#destroy'
end

7. Build Controllers
EmployeesController

class EmployeesController < ApplicationController
  before_action :authenticate_user!
  before_action :set_employee, only: [:show, :edit, :update, :destroy]

  def index
    @employees = Employee.all
  end

  def new
    @employee = Employee.new
  end

  def create
    @employee = Employee.new(employee_params)
    if @employee.save
      redirect_to employees_path, notice: 'Employee added successfully.'
    else
      render :new
    end
  end

  def edit; end

  def update
    if @employee.update(employee_params)
      redirect_to employees_path, notice: 'Employee updated successfully.'
    else
      render :edit
    end
  end

  def destroy
    @employee.destroy
    redirect_to employees_path, notice: 'Employee deleted successfully.'
  end

  private

  def set_employee
    @employee = Employee.find(params[:id])
  end

  def employee_params
    params.require(:employee).permit(:first_name, :last_name, :email, :phone, :position, :salary, :user_id, :department_id)
  end
end

8. Build Views

Use Bootstrap for UI. Example for app/views/employees/index.html.erb:

<h1>Employees</h1>
<table class="table">
  <thead>
    <tr>
      <th>Name</th>
      <th>Email</th>
      <th>Position</th>
      <th>Department</th>
      <th>Actions</th>
    </tr>
  </thead>
  <tbody>
    <% @employees.each do |employee| %>
      <tr>
        <td><%= employee.first_name + ' ' + employee.last_name %></td>
        <td><%= employee.email %></td>
        <td><%= employee.position %></td>
        <td><%= employee.department&.name %></td>
        <td>
          <%= link_to 'Edit', edit_employee_path(employee), class: 'btn btn-primary btn-sm' %>
          <%= link_to 'Delete', employee_path(employee), method: :delete, class: 'btn btn-danger btn-sm' %>
        </td>
      </tr>
    <% end %>
  </tbody>
</table>
<%= link_to 'Add New Employee', new_employee_path, class: 'btn btn-success' %>

9. Set Up Analytics Dashboard

Install and use chartkick for HR analytics:

<h1>HR Dashboard</h1>
<%= line_chart Employee.group_by_month(:created_at).count, title: 'Employees Joined Per Month' %>
<%= pie_chart Department.joins(:employees).group(:name).count, title: 'Employees by Department' %>

10. Deploy

Deploy your application on a cloud platform like Heroku:

heroku create hr-platform
git push heroku main
heroku run rails db:migrate

This code serves as a foundation for an HR platform. Additional features such as notifications, advanced analytics, or performance reviews can be added based on requirements.
